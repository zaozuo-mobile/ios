#IOS中实现多线程方式：
##NSThread
* 使用NSThread对象建立一个线程非常方便，但要使用NSThread管理多个线程较困难,不推荐使用;
* [NSThread currentThread]跟踪任务所在线程,适用于这三种技术。

```objective-c
- (void) testNSThread{
    // 创建方法1
    [NSThread detachNewThreadSelector:@selector(actionForNSThread:) toTarget:self withObject:@"1"];

    // 创建方法2
    NSThread *thread = [[NSThread alloc]initWithTarget:self selector:@selector(actionForNSThread:) object:@"1"];
    [thread start];
}

- (void) actionForNSThread:(NSString *)extraData{
    @autoreleasepool{
        NSLog(@"actionForNSThread===%@", extraData);
    }
}
```
##NSObject
#####NSObject的多线程方法：
* 开启后台执行任务的方法:
```objective-c    
- (void)performSelectorInBackground:(SEL)@Selector withObject:(id)arg
```
*  在后台线程中通知主线程执行任务的方法:
```objective-c
- (void)performSelectorOnMainThread:(SEL)@Selector withObject:(id)arg waitUntilDone:(BOOL)wait
```
#####NSObject的多线程方法注意事项:
 * NSObject的多线程方法使用的是NSThread的多线程技术.
 * NSThread的多线程技术不会自动使用@autoreleasepool.
	在使用NSObject或NSThread的多线程技术时,如果涉及到对象分配,需要手动添加@autoreleasepool.

##NSOperationQueue
* 支持线程池，可配置并发线程最大数量；
* 可设置线程间依赖关系，即在特定线程执行完毕后执行该线程；

```objective-c
- (void) testNSOperationQueue{
    // queue
    NSOperationQueue *myQueue = [[NSOperationQueue alloc] init];
    [myQueue setMaxConcurrentOperationCount:1];//设置并发线程数量.
    
    // function operation
    NSInvocationOperation *op = [[NSInvocationOperation alloc]
                                 initWithTarget:self selector:@selector(operationAction:) object:@(1)];
    // block operation
    NSBlockOperation *opBlock = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"NSBlockOperation----%@----obj",[NSThread currentThread]);
    }];
    
    [op addDependency:opBlock];//设置依赖关系（先后执行顺序）.
    
    [myQueue addOperation:op];
    [myQueue addOperation:opBlock];
}

-(void)operationAction:(id)obj{
    NSLog(@"NSInvocationOperation----%@----obj : %@ ",[NSThread currentThread], obj);
};
```

#线程同步
####NSLock
```objective-c
NSLock *lock = [[NSLock alloc]init];
[lock lock];// 加锁
[lock unlock];// 解锁
```
####@synchronized
同步代码块，将原子操作的代码放在synchronized中间。
```objective-c
@synchronized(self){     
}
```
####NSCondition
条件锁，除支持lock和unlock外，还支持暂停或恢复线程。
```objective-c
NSCondition *con = [[NSCondition alloc]init];
[con lock];// 加锁
[con unlock];// 解锁
[con wait]; // 暂停当前线程
[con signal]; // 恢复当前线程
```
####NSRecursiveLock：
递归锁，针对递归
####NSDistributedLock：
分布锁，它本身是一个互斥锁，基于文件方式实现锁机制，可以跨进程访问。
####pthread_mutex_t：
同步锁，基于C语言的同步锁机制，使用方法与其他同步锁机制类似。

