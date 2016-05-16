###Plist(NSArray/NSDictionary)：
> * 支持的数据类型有NSString、 NSNumber、NSDate、 NSArray、NSDictionary、BOOL、NSInteger、NSFloat等系统定义的数据类型，底层是基于key-value的NSDictionary。
* 项目中预置的plist文件只能读取不支持修改、删除；
* 运行期创建的plist文件支持读取、新写入、修改、删除等操作，写入时必须是完整的dic，不支持增量写入方式。

#####读取项目中预置plist文件：

```objective-c
- (void) testPlist{
    NSString *plistPath = [[NSBundle mainBundle] pathForResource:@"test" ofType:@"plist"];
    NSMutableDictionary *data = [[NSMutableDictionary alloc] initWithContentsOfFile:plistPath];
    NSLog(@"data===%@", data);
}
```
#####运行期创建的plist文件读取数据：

```objective-c
- (void) testReadPlist{
    // 存储路径
    NSArray *paths=NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask,YES);
    NSString *plistPath = [paths objectAtIndex:0];
    NSString *filename=[plistPath stringByAppendingPathComponent:@"test_new.plist"];
    // 查询
    NSMutableDictionary *data = [[NSMutableDictionary alloc] initWithContentsOfFile:filename];
    NSLog(@"%@", data);
}
```
#####运行期创建新plist并写入数据：

```objective-c
- (void) testWriteNewPlist{
    // 存储路径
    NSArray *paths=NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask,YES);
    NSString *plistPath = [paths objectAtIndex:0];
    NSString *filename=[plistPath stringByAppendingPathComponent:@"test_new.plist"];
    
    // 原始数据
    NSMutableDictionary *data = [NSMutableDictionary dictionary];
    // key存在value被覆盖，key不存在新增
    [data setObject:@(1) forKey:@"age"];

    // 写入数据
    BOOL success = [data writeToFile:filename atomically:YES];
    NSLog(@"success===%@", success?@"YES":@"NO");
    
    // 查询验证结果
    NSMutableDictionary *data1 = [[NSMutableDictionary alloc] initWithContentsOfFile:filename];
    NSLog(@"%@", data1);
}
```
#####运行期创建的plist修改、删除数据：
```objective-c
- (void) testModifyAndRemovePlist{
    // 存储路径
    NSArray *paths=NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask,YES);
    NSString *plistPath = [paths objectAtIndex:0];
    NSString *filename=[plistPath stringByAppendingPathComponent:@"test_new.plist"];
    
    NSMutableDictionary *data = [[NSMutableDictionary alloc] initWithContentsOfFile:filename];
    NSLog(@"%@", data);
    
    // 修改数据
    [data setObject:@(2) forKey:@"age"];
    // 删除数据
    [data removeObjectForKey:@"name"];
    
    // 写入数据
    BOOL success = [data writeToFile:filename atomically:YES];
    NSLog(@"success===%@", success?@"YES":@"NO");
    
    // 查询验证结果
    NSMutableDictionary *data1 = [[NSMutableDictionary alloc] initWithContentsOfFile:filename];
    NSLog(@"%@", data1);
}
```

###NSUserDefault：
> 支持的数据类型有NSString、 NSNumber、NSDate、 NSArray、NSDictionary、BOOL、NSInteger、NSFloat等系统定义的数据类型，如果要存放自定义的对象（如自定义的类对象），则必须将其转换成NSData存储。

```objective-c
- (void) testUserDefaults{
    NSUserDefaults *userDefaults = [NSUserDefaults standardUserDefaults];
    // 写入
    [userDefaults setInteger:2 forKey:@"age"];
    [userDefaults setObject:@"test_name" forKey:@"name"];
    // 强制写入磁盘
    [userDefaults synchronize];

    // 读取
    NSInteger age = [userDefaults integerForKey:@"age"];
    NSString *name = [userDefaults objectForKey:@"name"];
    NSLog(@"%ld===%@", age, name);
    
    //修改
    [userDefaults setInteger:3 forKey:@"age"];
    
    // 删除
    [userDefaults removeObjectForKey:@"name"];
    // 强制写入磁盘
    [userDefaults synchronize];
    
    NSLog(@"%ld===%@", [userDefaults integerForKey:@"age"],
          [userDefaults objectForKey:@"name"]);
}
```

###NSCoding：
可存储自定义对象，局限：一次性做读取和存储操作，不可局部增量操作

* entity实现NSCoding协议:

```objective-c
#import <Foundation/Foundation.h>
#import "ZZBaseEntity.h"

@interface TestUser : ZZBaseEntity<NSCoding>

@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger age;
@property (nonatomic, assign) NSInteger pid;

@end
```
* entity实现类重写encodeWithCoder、initWithCoder函数
实现序列化和反序列化。

```objective-c
#import "TestUser.h"

@implementation TestUser

- (void) setValue:(id)value forUndefinedKey:(NSString *)key{
    if ([key isEqualToString:@"id"]) {
         self.pid = [value integerValue];
    }
}

- (void)encodeWithCoder:(NSCoder *)aCoder{
    if (self.name){
        [aCoder encodeObject:self.name forKey:@"name"];
    }
    if (self.age){
        [aCoder encodeInteger:self.age forKey:@"age"];
    }
    if (self.pid){
        [aCoder encodeInteger:self.pid forKey:@"pid"];
    }
}

- (id)initWithCoder:(NSCoder *)aDecoder{
    if (self == [super init]) {
        self.name = [aDecoder decodeObjectForKey:@"name"];
        self.age = [aDecoder decodeIntegerForKey:@"age"];
        self.pid = [aDecoder decodeIntegerForKey:@"pid"];
    }
    return self;
}

- (NSString *)description{
    return [NSString stringWithFormat:@"%ld===%ld===%@", self.pid, self.age, self.name];
}
@end
```

* 保存对象到NSUserDefaults：

```objective-c
- (void) testUserDefaults{
    NSUserDefaults *userDefaults = [NSUserDefaults standardUserDefaults];
    // 写入
    TestUser *user = [[TestUser alloc]init];
    user.name = @"test_name";
    user.age = 2;
    
    NSData *data = [NSKeyedArchiver archivedDataWithRootObject:user];
    [userDefaults setObject:data forKey:@"user"];
    [userDefaults synchronize];
    
    // 读取
    NSData *data1 = [userDefaults objectForKey:@"user"];
    TestUser *user1 = [NSKeyedUnarchiver unarchiveObjectWithData:data1];
    NSLog(@"user1===%@", user1);
}
```
###Coredata：
> Coredata是对sqlite数据库ORM实现。

* NSManagedObjectContext
管理对象，上下文，持久性存储模型对象
* NSManagedObjectModel
数据模型，数据结构
* NSPersistentStoreCoordinator
连接数据库的
* NSManagedObject
数据记录
* NSFetchRequest
数据请求
* NSEntityDescription
表格实体结构
* .xcdatamodel文件编译后为.momd或者.mom文件
###Sqlite：
基本操作同Android中Sqlite。