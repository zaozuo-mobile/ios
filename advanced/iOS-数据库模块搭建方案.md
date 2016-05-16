>数据库作为App缓存设计的首选，存在一些开发的陷阱，同时需要考虑性能、开发效率和可维护性，笔者建议自行搭建数据库管理类，同时配合成熟的开源ORM框架快速搭建数据库模块。
本示例采用fmdb框架 https://github.com/ccgus/fmdb

## SQLite多线程访问问题分析：
```objective-c
- (void) testfmdb{
    // db path
    NSString * cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
    NSString * dbPath = [cachePath stringByAppendingPathComponent:@"test.sqlite"];
    
    // create db
    _db = [FMDatabase databaseWithPath:dbPath];
    
    // open db
    BOOL openDbResult = [_db open];
    NSLog(@"openDbResult===%@", openDbResult ? @"YES":@"NO");

    // create table
    BOOL createTableResult = [_db executeUpdate:@"create table if not exists user(id integer primary key autoincrement, name text, age integer)"];
    NSLog(@"createTableResult===%@", createTableResult ? @"YES":@"NO");
    
    // test multithreading
    NSOperationQueue *myQueue = [[NSOperationQueue alloc] init];
    [myQueue setMaxConcurrentOperationCount:10];
    for (int i = 0; i < 100; i ++) {
        NSBlockOperation *opBlock = [NSBlockOperation blockOperationWithBlock:^{
            [self insert];
            [self query];
        }];
        [myQueue addOperation:opBlock];
    }
    
}

-(void) insert{
    for (int i = 0; i < 100; i ++) {
        NSString *name = [NSString stringWithFormat:@"name_%d",i];
        NSString *age = [NSString stringWithFormat:@"%d",i];
        
        [_db executeUpdate:@"insert into user(name,age) values (?, ?)", name,age];
    }
}

-(void) query{
    FMResultSet * set = [_db executeQuery:@"select * from user"];
    while ([set next]) {
        int id = [set intForColumn:@"id"];
        NSString *name = [set stringForColumn:@"name"];
        int age = [set intForColumn:@"age"];
        NSLog(@"%d===%@===%d", id, name, age);
    }
}
```

* 调用testfmdb方法抛出异常：The FMDatabase is currently in use.
* ios中SQLite同Android中SQLite一样，数据库不支持多线程读写并发访问，Android底层对SQLite单个数据库连接读写操作做了同步处理，也仅能支持单数据库连接的并发访问。

## SQLite多线程访问解决方案：
###使用fmdb FMDatabaseQueue
* FMDatabaseQueue对所有数据库访问都在串行同步队列中执行，规避并发问题的产生；
* 所有数据库的操作，通过调用 inDatabase 在block回调中通过db访问数据库，block代码在当前线程中执行。

```objective-c
  -(void) testfmdb_queue{
    // db path
    NSString * cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
    NSString * dbPath = [cachePath stringByAppendingPathComponent:@"test.sqlite"];
    
    // create db and open db
    _queue = [FMDatabaseQueue databaseQueueWithPath:dbPath];
    
    // create table
    [_queue inDatabase:^(FMDatabase *db) {
        BOOL createTableResult = [_db executeUpdate:@"create table if not exists user(id integer primary key autoincrement, name text, age integer)"];
        NSLog(@"createTableResult===%@", createTableResult ? @"YES":@"NO");

    }];
    
    // test multithreading
    NSOperationQueue *myQueue = [[NSOperationQueue alloc] init];
    [myQueue setMaxConcurrentOperationCount:10];//设置并发线程数量.
    for (int i = 0; i < 20; i ++) {
        NSBlockOperation *opBlock = [NSBlockOperation blockOperationWithBlock:^{
            [self insert_queue];
            [self query_queue];
        }];
        [myQueue addOperation:opBlock];
    }
  }

  -(void) insert_queue{
    [_queue inDatabase:^(FMDatabase *db) {
        for (int i = 0; i < 20; i ++) {
            NSString *name = [NSString stringWithFormat:@"name_%d",i];
            NSString *age = [NSString stringWithFormat:@"%d",i];
            
            [db executeUpdate:@"insert into user(name,age) values (?, ?)", name,age];
        }
    }];
    
  }

  -(void) query_queue{
    [_queue inDatabase:^(FMDatabase *db) {
        FMResultSet * set = [db executeQuery:@"select * from user"];
        while ([set next]) {
            int id = [set intForColumn:@"id"];
            NSString *name = [set stringForColumn:@"name"];
            int age = [set intForColumn:@"age"];
            NSLog(@"%d===%@===%d", id, name, age);
        }
    }];
  }
```

## 配合fmdb快速搭建ORM：
* 定义DAO基类ZZBaseDAO；

```objective-c
#import <Foundation/Foundation.h>
#import "FMDatabase.h"
#import "FMDatabaseQueue.h"

@interface ZZBaseDAO : NSObject

@property(nonatomic,retain) FMDatabaseQueue * queue;
@property(nonatomic,copy) NSString * dbPath;

/**
 初始化数据库
 @params newDBPath 数据库完整路径
 **/
-(id) initWithDBPath:(NSString *) newDBPath;
/**
 获取数据库路径
 @params dbName 数据库名
 **/
- (NSString *) generateFilePath: (NSString *) dbName;

/**
 字典对象转为实体对象
 @params dict
 @params entity 实体数据，传入前需要创建好
 **/
+ (void) dictionaryToEntity:(NSDictionary *)dict entity:(NSObject*)entity;

/**
 实体对象转为字典对象，不支持对象中包含c基本数据类型，如：int、float等。
 @params entity
 **/
+ (NSDictionary *) entityToDictionary:(id)entity;

/**
 fmdb查询结果集转为实体对象数组
 @params set fmdb查询结果集
 @params clazz 实体数据，传入前需要创建好
 **/
+ (NSMutableArray *) dictionaryToEntityList:(FMResultSet *)set entity:(Class)clazz;

/**
 fmdb查询结果集转为实体对象数组，适用于查询结果为单条记录的情况
 @params set fmdb查询结果集
 @params clazz 实体数据，传入前需要创建好
 **/
+ (id) dictionaryToEntityOne:(FMResultSet *)set entity:(Class)clazz;

@end
```

* ZZBaseDAO实现类，通过调用NSObject的
setValuesForKeysWithDictionary和valueForKey实现NSDictionary和entity对象间转换；


```objective-c
#import "ZZBaseDAO.h"
#import "Constants.h"
#import <objc/runtime.h>

@implementation ZZBaseDAO

@synthesize queue, dbPath;

-(id) init{
    self = [super init];
    if (self) {
        dbPath = [self generateFilePath:DB_NAME_DEFAULT];
        queue = [FMDatabaseQueue databaseQueueWithPath: dbPath];
    }
    return self;
}

-(id) initWithDBPath:(NSString *) newDBPath{
    self = [super init];
    if (self) {
        // create db and open db
        dbPath = newDBPath;
        queue = [FMDatabaseQueue databaseQueueWithPath:newDBPath];
    }
    return self;
}

- (NSString *) generateFilePath: (NSString *) dbName{
    // db path
    NSString * cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
    NSString * newDBPath = [cachePath stringByAppendingPathComponent:dbName];
    return newDBPath;
}

+ (void) dictionaryToEntity:(NSDictionary *)dict entity:(NSObject*)entity{
    if (dict && entity) {
        [entity setValuesForKeysWithDictionary:dict];
    }
}

+ (NSDictionary *) entityToDictionary:(id)entity{
    u_int count;
    objc_property_t* properties = class_copyPropertyList([entity class], &count);
    
    NSMutableArray* propertyArray = [NSMutableArray array];
    NSMutableArray* valueArray = [NSMutableArray array];
    
    for (int i = 0; i < count; i++){
        // propertyNameStr
        objc_property_t prop = properties[i];
        const char* propertyName = property_getName(prop);
        NSString *propertyNameStr = [NSString stringWithUTF8String:propertyName];
        
        id value = [entity valueForKey:propertyNameStr];
        if(value != nil){
            // propertyArray
            [propertyArray addObject:propertyNameStr];
            // valueArray
            [valueArray addObject:value];
        }
    }

    free(properties);
    
    // entity -> dict
    NSDictionary* returnDic = [NSDictionary dictionaryWithObjects:valueArray forKeys:propertyArray];
    
    return returnDic;
}

+ (NSMutableArray *) dictionaryToEntityList:(FMResultSet *)set entity:(Class)clazz{
    if (set && clazz) {
        NSMutableArray * arr = [NSMutableArray array];
        while ([set next]) {
            NSDictionary *dic = [set resultDictionary];
            id entity = [[clazz alloc]init];
            [ZZBaseDAO dictionaryToEntity:dic entity:entity];
            [arr addObject:entity];
        }
        return arr;
    }
    return nil;
}

+ (id) dictionaryToEntityOne:(FMResultSet *)set entity:(Class)clazz{
    if (set && clazz) {
        [set next];
        NSDictionary *dic = [set resultDictionary];
        id entity = [[clazz alloc]init];
        [ZZBaseDAO dictionaryToEntity:dic entity:entity];
        return entity;
    }
    return nil;
}

@end
```
    
* fmdb数据库操作ORM实例：

```objective-c
-(void) testfmdb_queue{
   _queue = [self getFMDatabaseQueue];
    
    // create table
    [_queue inDatabase:^(FMDatabase *db) {
        BOOL createTableResult = [_db executeUpdate:@"create table if not exists user(id integer primary key autoincrement, name text, age integer)"];
        NSLog(@"createTableResult===%@", createTableResult ? @"YES":@"NO");
        
    }];
    
    // test multithreading
    NSOperationQueue *myQueue = [[NSOperationQueue alloc] init];
    [myQueue setMaxConcurrentOperationCount:10];//设置并发线程数量.
    for (int i = 0; i < 1; i ++) {
        NSBlockOperation *opBlock = [NSBlockOperation blockOperationWithBlock:^{
            [self insert_queue];
            [self query_queue];
        }];
        [myQueue addOperation:opBlock];
    }
}

-(void) insert_queue{
    [_queue inDatabase:^(FMDatabase *db) {
        for (NSInteger i = 0; i < 20; i ++) {
            TestUser *user = [[TestUser alloc]init];
            user.name = [NSString stringWithFormat:@"test_name_%ld",i];
            user.age = i;
            NSDictionary *dic = [ZZBaseDAO entityToDictionary:user];
            NSLog(@"dic===%@", dic);
            
            BOOL insertResult = [db executeUpdate:@"insert into user(name,age) values (:name, :age)" withParameterDictionary:dic];
            NSLog(@"insertResult===%@", insertResult?@"YES":@"NO");
        }
    }];
    
}

-(void) query_queue{
    [_queue inDatabase:^(FMDatabase *db) {
        FMResultSet *set = nil;
        @try {
            set = [db executeQuery:@"select id,name,age from user limit 10"];
            NSMutableArray *arr = [ZZBaseDAO dictionaryToEntityList:set entity:[TestUser class]];
            if (arr) {
                for (TestUser* user in arr) {
                    NSLog(@"%ld===%@===%ld", user.pid, user.name, (long)user.age);
                }
            }
        }
        @catch (NSException *exception) {
        }
        @finally {
            if (set) {
                [set close];
            }
        }
    }];
}
```

* 处理数据库中特殊字段如id等与oc关键字冲突的情况：
定义函数setValue:forUndefinedKey，实现特殊数据库字段与entity中property的映射。

```objective-c
#import <Foundation/Foundation.h>

@interface ZZBaseEntity : NSObject

- (void) setValue:(id)value forUndefinedKey:(NSString *)key;

@end


#import "ZZBaseEntity.h"

@implementation ZZBaseEntity

- (void) setValue:(id)value forUndefinedKey:(NSString *)key{
//    if ([key isEqualToString:@"id"]) {
//        self.pid = [value integerValue];
//    }
}
@end
```

 ##使用fmdb升级数据库：
* 通过判断表中字段是否存在来确定是否需要升级

```objective-c
// create db and open db
_queue = [FMDatabaseQueue databaseQueueWithPath:dbPath];

[_queue inDatabase:^(FMDatabase *db) {
    // 判断数据库字段是否存在,需要#import"FMDatabaseAdditions.h"
    BOOL exist = [db columnExists:@"name" inTableWithName:@"user"];
    NSLog(@"exist===%@", exist?@"YES":@"NO");
    
}];
```


