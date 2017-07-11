---
title: iOS10系统崩溃问题修复——NSURLSessionTaskMetrics
date: 2017-07-10 15:47:52
tags: 
	- iOS
	- 10.2系统崩溃
	- NSURLSessionTaskMetrics
---



## 现象

奔溃日志：

```objective-c
CoreFoundation    0x1816721d8    ___exceptionPreprocess + 148

libobjc.A.dylib    0x1800ac55c    _objc_exception_throw

Foundation    0x182280b4c    -[_NSConcreteDateInterval dealloc] + 0

CFNetwork    0x181eab3e0    -[__NSCFURLSessionTaskMetrics _initWithTask:] + 692

CFNetwork    0x181eab028    -[NSURLSessionTaskMetrics _initWithTask:] + 100

CFNetwork    0x181d3016c    -[__NSCFURLLocalSessionConnection _tick_finishing] + 372

libdispatch.dylib    0x1804fd200    __dispatch_call_block_and_release + 24

libdispatch.dylib    0x1804fd1c0    __dispatch_client_callout + 16

libdispatch.dylib    0x18050b444    __dispatch_queue_serial_drain + 928

libdispatch.dylib    0x1805009a8    __dispatch_queue_invoke + 652

libdispatch.dylib    0x18050d38c    __dispatch_root_queue_drain + 572

libdispatch.dylib    0x18050d0ec    __dispatch_worker_thread3 + 124

libsystem_pthread.dylib    0x1807062b8    __pthread_wqthread + 1288

libsystem_pthread.dylib    0x180705da4    _start_wqthread + 4
```

`NSURLSessionTaskMetrics`这个类在 iOS10.0新增。此崩溃发生在iOS10.0 - 10.2，不包括 iOS10.2。

在[Open Radar](http://openradar.appspot.com/28301343)上找到了这个 bug report。

```
NSGenericException "Start date cannot be later in time than end date!" in -[NSURLSessionTaskMetrics _initWithTask:]
Originator:	nobrien	
Number:	rdar://28301343	Date Originated:	14-Sep-2016 09:16 AM
Status:	Open	Resolved:	
Product:	iOS SDK	Product Version:	10
Classification:	Crash/Hang/Data Loss	Reproducible:	Unable
 
Summary:
Trickling in crashes with NSGenericException - Start date cannot be later in time than end date!

Steps to Reproduce:
Cannot reproduce locally

Expected Results:
No exception

Actual Results:
Exception

Regression:
New to iOS 10 (makes sense since this is in the new NSURLSessionTaskMetrics code

Notes:
Is there potential risk with clock skew?  FWIW: we do all our timing metrics with mach time to avoid potential issues with timezone traversals, regional settings changes or daylight savings time changes.
```

同时在评论中，网友给出了临时的解决办法，那么我们就参考这个方法来修复我们的 bug。

解决方案：

 hook`NSURLSessionTaskMetrics`的`_initWithTask:`函数。

### 本地代码修复

代码如下：

```objective-c
@interface NSURLSessionTask(DDBugFix)
@property double startTime;
@end

@interface NSURLSessionTaskMetrics(DDBugFix)
- (instancetype)DDBugFix_initWithTask:(NSURLSessionTask *)task;
@end

@implementation NSURLSessionTaskMetrics(DDBugFix)
- (instancetype)DDBugFix_initWithTask:(NSURLSessionTask *)task
{
    if ([NSDate timeIntervalSinceReferenceDate] - task.startTime < 0)
    {
        CFRelease((__bridge CFTypeRef)self);
        return nil;
    }
    
    return [self DDBugFix_initWithTask:task];
}
@end

@implementation DDSystemBugFix
+(void)load {
    NSString * systemVersion = [[UIDevice currentDevice] systemVersion];
    NSString * startVersion = @"10.0";
    NSString * endVersion = @"10.2";
    if ([systemVersion compare:startVersion options:NSNumericSearch] >= NSOrderedSame &&
        [systemVersion compare:endVersion options:NSNumericSearch] < NSOrderedSame) {
        DDSwizzleInstanceMethod([NSURLSessionTaskMetrics class],@selector(_initWithTask:),@selector(DDBugFix_initWithTask:));
    }
}
@end
```

写完之后感觉也没什么问题，但是后来同事提醒`@selector(_initWithTask:)`这个方式在提审的时候是不是有判断为调用私有 API 的风险。

后修改如下：

```objective-c
@interface NSURLSessionTaskMetrics(DDBugFix)
- (instancetype)DDBugFix_initWithTask:(NSURLSessionTask *)task;
@end

@implementation NSURLSessionTaskMetrics(DDBugFix)
- (instancetype)DDBugFix_initWithTask:(NSURLSessionTask *)task
{
    NSNumber * timeNumb = [task valueForKey:@"startTime"];
    if ([NSDate timeIntervalSinceReferenceDate] - [timeNumb doubleValue] < 0)
    {
        CFRelease((__bridge CFTypeRef)self);
        return nil;
    }
    
    return [self DDBugFix_initWithTask:task];
}
@end

@implementation DDSystemBugFix
+(void)load {
    NSString * systemVersion = [[UIDevice currentDevice] systemVersion];
    NSString * startVersion = @"10.0";
    NSString * endVersion = @"10.2";
    if ([systemVersion compare:startVersion options:NSNumericSearch] >= NSOrderedSame &&
        [systemVersion compare:endVersion options:NSNumericSearch] < NSOrderedSame) {
        DDSwizzleInstanceMethod([NSURLSessionTaskMetrics class],NSSelectorFromString(@"_initWithTask:"),@selector(DDBugFix_initWithTask:));
    }
}
@end
```



### JSPatch下修复

```javascript
require("UIDevice, NSDate");

defineClass("NSURLSessionTaskMetrics", {
    __initWithTask: function(task) {
        var systemVersion = UIDevice.currentDevice().systemVersion();
        var startVersion = "10.0";
        var endVersion = "10.2";
        if (systemVersion.compare_options(startVersion, 64) >= 0 && systemVersion.compare_options(endVersion, 64) < 0) {
            var timeNumb = task.valueForKey("startTime");
            if (NSDate.timeIntervalSinceReferenceDate() - timeNumb < 0) {
                return null;
            }
        }
        return self.ORIG__initWithTask(task);
    }
}, {});
```

问题：由于JSPatch缺少 CFRelease 的转换，导致内存泄漏。但是内存泄露总比 crash 好。

那么 JSPatch是否提供了调用 C 函数的解决方案呢？

[C 函数调用](https://github.com/bang590/JSPatch/wiki/C-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8)

其中内存相关的，单独封装在了一个 JPMemory类中

[JPMemory 使用文档](https://github.com/bang590/JSPatch/wiki/JPMemory-%E4%BD%BF%E7%94%A8%E6%96%87%E6%A1%A3)

JSPatch 的扩展使用都需要将特定的扩展类加入的项目中，支持 cocoapods

```ruby
pod 'JSPatch'
pod 'JSPatch/Extensions'
pod 'JSPatch/JPCFunction'
```

由于之前版本中没有引入相关源码，无法使用 CFRelease 函数，但是在之后的版本中可以考虑引入。



参考文档：

[http://openradar.appspot.com/28301343](http://openradar.appspot.com/28301343)

https://github.com/bang590/JSPatch/wiki