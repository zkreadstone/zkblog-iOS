NSTimer 会发生循环引用，如何避免，可以使用中间者

```Objective-c

@interface ZKTimerProxy : NSProxy
+ (instancetype)proxyWithTarget:(id)target;
@end

```

```Objective-c

#import "ZKTimerProxy.h"
@interface ZKTimerProxy()
@property (nonatomic, weak)id target;
@end
@implementation ZKTimerProxy
+ (instancetype)proxyWithTarget:(id)target
{
    ZKTimerProxy *proxy = [self alloc];
    proxy.target = target;
    return  proxy;
}
//查找方法的签名
-(NSMethodSignature *)methodSignatureForSelector:(SEL)sel
{
    return  [self.target methodSignatureForSelector:sel];
}
//重定向
-(void)forwardInvocation:(NSInvocation *)invocation
{
    [invocation invokeWithTarget:self.target];
}
@end


```

使用方式：
```Objective-c
  ZKTimerProxy *proxy = [ZKTimerProxy proxyWithTarget:self];
  self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:proxy selector:@selector(test) userInfo:nil repeats:YES];
```

避免计时器 循环引用的方式  <img width="437" alt="image" src="https://user-images.githubusercontent.com/17443960/128119847-e551d148-41ca-43df-9bdb-fcb50970866e.png">

