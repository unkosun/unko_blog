---
title: Xcode断点调试
date: 2019-01-11 14:41:19
tags:
    - iOS
---

在我们的日常开发过程常常会遇到一些莫名其妙的问题，如何定位这些问题呢？就需要调试了。其中断点调试是我们日常开发过程中用到得最最常用的手段。



## 断点介绍



断点类型

1. 一般断点
2. 特殊断点



### 一、 一般断点

一般断点：通过在代码行号前点击鼠标或者快捷键“command+\”打下的断点。



#### 条件断点

给断点设置一个条件，当条件满足后，断点才生效。



```objective-c
- (void)testConditionBreakPoint {
    for (int i = 0; i<100; ++i) {
        NSLog(@"%d",i);
    }
}
```



![image-20180625111748528](https://ws4.sinaimg.cn/large/006tKfTcgy1fsodubfbc9j30qg0840vk.jpg)



上图中的条件断点，余数为1的断点都会生效。

#### 断点编辑

**Condition**:指的是条件表达式，该项允许我们对断点生效设置条件，表示当满足某一特定条件的前提下，该断点才生效。（该条件的录入，不能够识别预处理的宏定义，也不能识别断点作用域之外的变量和方法）。

**Ignore**：忽略次数。它指定了在断点生效，应用暂停之前，代码忽略断点的次数。你如果希望应用运行一段时间后断点才生效，那么就可以使用这个选项。比如说在调试某一循环体的时候。

**Action**:动作。它表示当断点生效时，Xcode作出反应后的行为动作。

![image-20180626091237553](https://ws1.sinaimg.cn/large/006tKfTcgy1fsoduh5e79j30pu09ijty.jpg)

如上图，我们可以看到6种Action，接下来我们来介绍下常用的几种：

**Sound**：断点触发是会发出声音提醒。

**Capture GPU Frame** ：当断点生效时，捕获GPU当前所绘制的帧。该功能是辅助图形调试的。平常很少用到，不是很了解，就不详细介绍了

**AppleScript**: AppleScript是苹果提供的一种脚本语言，用来执行一些预先指定的行为。

**Shell Command**: 在断点生效的时候，执行一段shell脚本。

**Debugger Command**:

可以在断点中使用lldb命令，例如：`po i` ,`expr NSLog(@"Ok, print a log: %d",i)`,`bt`

**Log Message**: 打印日志，可以将日志输出到控制台，或者播报。



此处看demo代码



#### 组合使用

Condition和Ignore是否可以一起使用，而同时生效？





#### 二、特殊断点

#### 1.  异常断点（Exception Breakpoint）

![image-20180627142419767](https://ws2.sinaimg.cn/large/006tKfTcgy1fsppvnsjjmj30qg07ygmz.jpg)

**Exception** :可以让你选择响应Objective－C对象抛出的异常，也可以选择响应C++对象抛出的异常。

**Break**:选择断点所接收的异常，是接收`Throw`语句抛出的异常还是`Catch`语句的

```objective-c
- (void)testExceptionOnThrowBreakPoint {
    NSException * exception = [NSException exceptionWithName:@"自定义异常" reason:@"我就想抛异常" userInfo:nil];
    @throw exception;
}
```

我们选择Break为 **On Throw**,可以看到异常断点生效，在左侧的调用堆栈中可以看到`objc_exception_throw`

![image-20180627150336155](https://ws1.sinaimg.cn/large/006tKfTcgy1fspr0fztokj30gm0mg429.jpg)

如果我们将Break设置为**On Catch**时，就无法捕获到异常。

#### 符号断点（Symbolic Breakpoint）

他可以中断某个方法的调用，可谓是异常强大，在断点导航器界面，点击＋号，选择Symbolic Breakpoint选项。

大家可以看到它比普通断点的自定义设置界面多出了两个内容，其一是**Symbol**，他用来设置当前断点作用域所能识别的方法，这里面既可以是自定义的方法，也可以是系统的API方法。（注意必须表明是类方法还是成员方法）

另一个**Module**是模块的意思，用于模块筛选，可以避免不同库中方法名或者函数名相同。

 ![symbolic breakpoint](https://ws3.sinaimg.cn/large/006tKfTcgy1fsqtgmb427j30p409atb0.jpg)

 

 

大家在写UI代码做约束布局的时候，经常可以再控制台看到如下日志



```
2018-07-03 08:58:32.876523+0800 同城游[1142:1418156] [LayoutConstraints] Unable to simultaneously satisfy constraints.
	Probably at least one of the constraints in the following list is one you don't want. 
	Try this: 
		(1) look at each constraint and try to figure out which you don't expect; 
		(2) find the code that added the unwanted constraint or constraints and fix it. 
(
    "<MASLayoutConstraint:0x1c40afde0 UIButton:0x10940a0f0.left == GCAddFriendHeaderView:0x109409cf0.left + 12>",
    "<MASLayoutConstraint:0x1c40afe40 UIButton:0x10940a0f0.right == GCAddFriendHeaderView:0x109409cf0.right - 12>",
    "<NSLayoutConstraint:0x1d04863b0 GCAddFriendHeaderView:0x109409cf0.width == 0>"
)

Will attempt to recover by breaking constraint 
<MASLayoutConstraint:0x1c40afe40 UIButton:0x10940a0f0.right == GCAddFriendHeaderView:0x109409cf0.right - 12>

Make a symbolic breakpoint at UIViewAlertForUnsatisfiableConstraints to catch this in the debugger.
The methods in the UIConstraintBasedLayoutDebugging category on UIView listed in <UIKit/UIView.h> may also be helpful.
```

 

那么我们按照提示来增加一个符号断点：UIViewAlertForUnsatisfiableConstraints

![UIViewAlertForUnsatisfiableConstraints](https://upload-images.jianshu.io/upload_images/1613923-f13b2dd8c4806b65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/519)

同时设置**Action**为**Debugger Command**, ` po [[UIWindow keyWindow] _autolayoutTrace]`

使用` po [[UIWindow keyWindow] _autolayoutTrace]`可以将window中的自动布局的view层级打印出来

```
•UIWindow:0x1091e7140 - AMBIGUOUS LAYOUT
|   •UILayoutContainerView:0x11890efb0
|   |   UINavigationTransitionView:0x118912e90
|   |   |   UIViewControllerWrapperView:0x118924930
|   |   |   |   UIView:0x10951a8c0
|   |   |   |   |   UILayoutContainerView:0x1090120f0
|   |   |   |   |   |   UITransitionView:0x108605aa0
|   |   |   |   |   |   |   UIViewControllerWrapperView:0x10860cc50
|   |   |   |   |   |   |   |   •UIView:0x10950fbb0
|   |   |   |   |   |   |   |   |   +GCNavSegmentedView:0x10950fd90
|   |   |   |   |   |   |   |   |   |   +UIView:0x109510890
|   |   |   |   |   |   |   |   |   |   |   +UIButton:0x109510b40'消息'
|   |   |   |   |   |   |   |   |   |   |   |   UIButtonLabel:0x109510e60'消息'
|   |   |   |   |   |   |   |   |   |   |   +UIButton:0x109511150'好友'
|   |   |   |   |   |   |   |   |   |   |   |   UIButtonLabel:0x109511470'好友'
|   |   |   |   |   |   |   |   |   |   |   UIView:0x109511760
|   |   |   |   |   |   |   |   |   UIScrollView:0x10d81ea00
|   |   |   |   |   |   |   |   |   |   •UIView:0x109512720
|   |   |   |   |   |   |   |   |   |   |   *<UILayoutGuide: 0x1cc1a2220 - "UIViewSafeAreaLayoutGuide", layoutFrame = {{0, 0}, {375, 596}}, owningView = <UIView: 0x109512720; frame = (0 0; 375 596); autoresize = H; layer = <CALayer: 0x1c0030680>>>
|   |   |   |   |   |   |   |   |   |   |   *UIView:0x1090143d0
|   |   |   |   |   |   |   |   |   |   |   |   *UITableView:0x10d822400
|   |   |   |   |   |   |   |   |   |   |   |   |   GCNotificationListCell:0x10a20b800'GCNotificationListCell'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   •UITableViewCellContentView:0x113b346b0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *GCAvatarFrameView:0x113b607b0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b9aa10
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x109165a60
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b67e50
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b6d340'平台公告'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x109174f60'6月22日'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x1091faf10'等待'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b554d0
|   |   |   |   |   |   |   |   |   |   |   |   |   GCNotificationListCell:0x10a1cc800'GCNotificationListCell'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   •UITableViewCellContentView:0x1091dfa30
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *GCAvatarFrameView:0x113b29070
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b31a00
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x113b5e610
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b23a90
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x1091e2750'帐号测试02'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b4c840'2018/06/27'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x1091605c0'图'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b41970
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x1091077a0
|   |   |   |   |   |   |   |   |   |   |   |   |   UIView:0x10950f370
|   |   |   |   |   |   |   |   |   |   |   |   |   UIView:0x109510690
|   |   |   |   |   |   |   |   |   |   |   |   |   UIImageView:0x113b4d990
|   |   |   |   |   |   |   |   |   |   |   |   |   UIView:0x113b88820
|   |   |   |   |   |   |   |   |   |   |   |   |   UIImageView:0x113b4d720
|   |   |   |   |   |   |   |   |   |   •UIView:0x10860a350
|   |   |   |   |   |   |   |   |   |   |   *<UILayoutGuide: 0x1c81a1ea0 - "UIViewSafeAreaLayoutGuide", layoutFrame = {{0, 0}, {375, 596}}, owningView = <UIView: 0x10860a350; frame = (375 0; 375 596); autoresize = H; layer = <CALayer: 0x1c8028b40>>>
|   |   |   |   |   |   |   |   |   |   |   *UIView:0x10860af00
|   |   |   |   |   |   |   |   |   |   |   |   *UITableView:0x108813e00
|   |   |   |   |   |   |   |   |   |   |   |   |   •GCFriendListCell:0x10a233a00'GCFriendListCell'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   +UITableViewCellContentView:0x113b932b0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b912f0'咋个家常菜'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b93830'离线'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIButton:0x113b97f90'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   UIButtonLabel:0x113b9b750'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *GCAvatarFrameView:0x113b93cf0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b95d00
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x113b92850
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b94330
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x113b9ca20
|   |   |   |   |   |   |   |   |   |   |   |   |   |   _UITableViewCellSeparatorView:0x113b92cb0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b9ba80'已求过'
|   |   |   |   |   |   |   |   |   |   |   |   |   •GCFriendListCell:0x10a204000'GCFriendListCell'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   +UITableViewCellContentView:0x113b84200
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b3c890'zzkk'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b3ce10'离线'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIButton:0x113b84740'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   UIButtonLabel:0x113b8c5e0'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *GCAvatarFrameView:0x113b84c00
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b86c10
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x113b7d5a0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b7d200
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x113b8ae60
|   |   |   |   |   |   |   |   |   |   |   |   |   |   _UITableViewCellSeparatorView:0x113b83c90
|   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b8c910'已求过'
|   |   |   |   |   |   |   |   |   |   |   |   |   •GCFriendListCell:0x10a210e00'GCFriendListCell'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   +UITableViewCellContentView:0x109188840
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x109161250'zzff'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x109187d00'离线'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIButton:0x113b7c160'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   UIButtonLabel:0x113b5aa90'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *GCAvatarFrameView:0x113b5e3b0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b5b1f0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x10918de00
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b50740
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x113b53600
|   |   |   |   |   |   |   |   |   |   |   |   |   |   _UITableViewCellSeparatorView:0x10915e530
|   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b52c30'已求过'
|   |   |   |   |   |   |   |   |   |   |   |   |   •GCFriendListCell:0x10a209e00'GCFriendListCell'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   +UITableViewCellContentView:0x109165ee0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b57fc0'杨鑫誉'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x109166520'离线'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIButton:0x113b57640'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   UIButtonLabel:0x113b3f7a0'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *GCAvatarFrameView:0x113b57960
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x1091099e0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x109166800
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b572a0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x113b40a70
|   |   |   |   |   |   |   |   |   |   |   |   |   |   _UITableViewCellSeparatorView:0x113b52710
|   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b3fad0'已求过'
|   |   |   |   |   |   |   |   |   |   |   |   |   •GCFriendListCell:0x10a038800'GCFriendListCell'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   +UITableViewCellContentView:0x113b1f9b0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b7b740'邬翰羿'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x1091efca0'离线'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIButton:0x113b82ab0'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   UIButtonLabel:0x113b51dd0'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *GCAvatarFrameView:0x113b7f5f0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b5ce50
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x109166f20
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b61840
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x10915e110
|   |   |   |   |   |   |   |   |   |   |   |   |   |   _UITableViewCellSeparatorView:0x113b3bb10
|   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b28c90'已求过'
|   |   |   |   |   |   |   |   |   |   |   |   |   •GCFriendListCell:0x10a1d8c00'GCFriendListCell'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   +UITableViewCellContentView:0x113b6bc70
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x109184ba0'裘糜冰珍'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x10910d050'离线'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIButton:0x113b1d3a0'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   UIButtonLabel:0x113b1cf90'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *GCAvatarFrameView:0x109174ac0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x113b1cc10
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x109181e20
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x109184480
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x109183880
|   |   |   |   |   |   |   |   |   |   |   |   |   |   _UITableViewCellSeparatorView:0x113b115c0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b288b0'已求过'
|   |   |   |   |   |   |   |   |   |   |   |   |   •GCFriendListCell:0x10a839e00'GCFriendListCell'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   +UITableViewCellContentView:0x109230150
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x1092dd380'萧山晴霞'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x10923b380'台州麻将 游戏中'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIButton:0x10923b660'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   UIButtonLabel:0x11891e9e0'跟随'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *GCAvatarFrameView:0x10923b980
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x109204db0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x109204fe0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x1092bd930
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x118922500
|   |   |   |   |   |   |   |   |   |   |   |   |   |   _UITableViewCellSeparatorView:0x1092bdb60
|   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x1092d5c60'已求过'
|   |   |   |   |   |   |   |   |   |   |   |   |   GCAddFriendTableViewCell:0x10aa46800'GCAddFriendTableViewCell_...'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   •UITableViewCellContentView:0x1092e5c60
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x118924620
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x11891bf50'新的好友'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIImageView:0x1189817d0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x10923a360
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x1189812f0
|   |   |   |   |   |   |   |   |   |   |   |   |   UIView:0x118903660
|   |   |   |   |   |   |   |   |   |   |   |   |   •UIView:0x113b7cca0
|   |   |   |   |   |   |   |   |   |   |   |   |   |   *UILabel:0x113b9a550'我的好友'
|   |   |   |   |   |   |   |   |   |   |   |   |   |   *UIView:0x113b9d930
|   |   |   |   |   |   |   |   |   |   |   |   |   UIImageView:0x113b623f0
|   |   |   |   |   |   |   |   |   |   |   |   |   UIImageView:0x113b630f0
|   |   |   |   |   |   |   |   |   |   UIImageView:0x113b4ebc0
|   |   |   |   |   |   UITabBar:0x1090122f0
|   |   |   |   |   |   |   _UIBarBackground:0x108605f90
|   |   |   |   |   |   |   |   UIImageView:0x1086061f0
|   |   |   |   |   |   |   |   UIImageView:0x108606420
|   |   |   |   |   |   |   UITabBarButton:0x1086076f0
|   |   |   |   |   |   |   |   UITabBarSwappableImageView:0x108607c00
|   |   |   |   |   |   |   UITabBarButton:0x108608050
|   |   |   |   |   |   |   |   UITabBarSwappableImageView:0x108608560
|   |   |   |   |   |   |   UITabBarButton:0x1086087b0
|   |   |   |   |   |   |   |   UITabBarSwappableImageView:0x108608ac0
|   |   |   |   |   |   |   UITabBarButton:0x109407cc0
|   |   |   |   |   |   |   |   UITabBarSwappableImageView:0x109407fd0
|   |   |   |   |   |   |   UIView:0x113b0cbb0
|   |   |   |   |   |   |   UIView:0x113b17330
|   |   |   |   |   |   UINavigationBar:0x109513fb0
|   |   |   |   _UIParallaxDimmingView:0x10940c0b0
|   |   |   |   UIView:0x10951a6e0
|   |   |   |   |   _UIParallaxDimmingView:0x10940c4b0
|   |   |   |   |   |   UIImageView:0x10940ca70
|   |   |   |   |   |   •UIView:0x108602400, MISSING HOST CONSTRAINTS
|   |   |   |   |   |   |   *<UILayoutGuide: 0x1c81a1ce0 - "UIViewSafeAreaLayoutGuide", layoutFrame = {{0, 88}, {375, 690}}, owningView = <UIView: 0x108602400; frame = (0 0; 375 812); autoresize = W+H; layer = <CALayer: 0x1c8027ca0>>>- AMBIGUOUS LAYOUT for UILayoutGuide:0x1c81a1ce0'UIViewSafeAreaLayoutGuide'.minX{id: 4441}, UILayoutGuide:0x1c81a1ce0'UIViewSafeAreaLayoutGuide'.minY{id: 4438}, UILayoutGuide:0x1c81a1ce0'UIViewSafeAreaLayoutGuide'.Width{id: 4448}, UILayoutGuide:0x1c81a1ce0'UIViewSafeAreaLayoutGuide'.Height{id: 4445}
|   |   |   |   |   |   |   *UIView:0x10860e090
|   |   |   |   |   |   |   |   *UITableView:0x10d014c00
|   |   |   |   |   |   |   |   |   UIView:0x109409b10
|   |   |   |   |   |   |   |   |   •GCAddFriendHeaderView:0x109409cf0
|   |   |   |   |   |   |   |   |   |   *UIButton:0x10940a0f0' 请输入同城游序号/昵称/手机号'- AMBIGUOUS LAYOUT for UIButton:0x10940a0f0' 请输入同城游序号/昵称/手机号'.minY{id: 4475}
|   |   |   |   |   |   |   |   |   |   |   UIImageView:0x1092c3580
|   |   |   |   |   |   |   |   |   |   |   UIButtonLabel:0x10940a410
|   |   |   |   |   |   |   |   |   UIImageView:0x109516990
|   |   |   |   |   |   |   |   |   UIImageView:0x109516bc0
|   |   +UINavigationBar:0x11890f1b0
|   |   |   _UIBarBackground:0x11890f470
|   |   |   |   UIImageView:0x11890f6d0
|   |   |   |   UIImageView:0x11890f900
|   |   |   _UINavigationBarLargeTitleView:0x118910210'添加好友'
|   |   |   |   UILabel:0x118910730
|   |   |   |   UILabel:0x113bb9bb0
|   |   |   •_UINavigationBarContentView:0x11890ff60'添加好友'
|   |   |   |   *<UILayoutGuide: 0x1d09bd260 - "BackButtonGuide(0x11890ee00)", layoutFrame = {{0, 0}, {8, 44}}, owningView = <_UINavigationBarContentView: 0x11890ff60; frame = (0 0; 375 44); layer = <CALayer: 0x1d063df80>>>
|   |   |   |   *<UILayoutGuide: 0x1d09bd0a0 - "LeadingBarGuide(0x11890ee00)", layoutFrame = {{8, 0}, {0, 44}}, owningView = <_UINavigationBarContentView: 0x11890ff60; frame = (0 0; 375 44); layer = <CALayer: 0x1d063df80>>>
|   |   |   |   *<UILayoutGuide: 0x1d09bcfc0 - "TitleView(0x11890ee00)", layoutFrame = {{8, 0}, {334, 44}}, owningView = <_UINavigationBarContentView: 0x11890ff60; frame = (0 0; 375 44); layer = <CALayer: 0x1d063df80>>>
|   |   |   |   *<UILayoutGuide: 0x1d09bcee0 - "TrailingBarGuide(0x11890ee00)", layoutFrame = {{342, 0}, {17, 44}}, owningView = <_UINavigationBarContentView: 0x11890ff60; frame = (0 0; 375 44); layer = <CALayer: 0x1d063df80>>>
|   |   |   |   *<UILayoutGuide: 0x1d09bce00 - "UIViewLayoutMarginsGuide", layoutFrame = {{16, 0}, {343, 44}}, owningView = <_UINavigationBarContentView: 0x11890ff60; frame = (0 0; 375 44); layer = <CALayer: 0x1d063df80>>>
|   |   |   |   *<UILayoutGuide: 0x1c01a1500 - "BackButtonGuide(0x109518800)", layoutFrame = {{0, 0}, {16, 44}}, owningView = <_UINavigationBarContentView: 0x11890ff60; frame = (0 0; 375 44); layer = <CALayer: 0x1d063df80>>>
|   |   |   |   *<UILayoutGuide: 0x1c01a1ce0 - "LeadingBarGuide(0x109518800)", layoutFrame = {{16, 0}, {44, 44}}, owningView = <_UINavigationBarContentView: 0x11890ff60; frame = (0 0; 375 44); layer = <CALayer: 0x1d063df80>>>
|   |   |   |   *<UILayoutGuide: 0x1c01a1dc0 - "TitleView(0x109518800)", layoutFrame = {{60, 0}, {307, 44}}, owningView = <_UINavigationBarContentView: 0x11890ff60; frame = (0 0; 375 44); layer = <CALayer: 0x1d063df80>>>
|   |   |   |   *<UILayoutGuide: 0x1c01a1ea0 - "TrailingBarGuide(0x109518800)", layoutFrame = {{367, 0}, {0, 44}}, owningView = <_UINavigationBarContentView: 0x11890ff60; frame = (0 0; 375 44); layer = <CALayer: 0x1d063df80>>>
|   |   |   |   *_UIButtonBarStackView:0x113b5df90
|   |   |   |   |   *<UILayoutGuide: 0x1d41a86c0 - "UIButtonBar.flexibleSpaceEqualSizeLayoutGuide", layoutFrame = {{-16, 0}, {8, 0}}, owningView = <_UIButtonBarStackView: 0x113b5df90; frame = (16 0; 44 44); alpha = 0; layer = <CALayer: 0x1d422e560>>>- AMBIGUOUS LAYOUT for UILayoutGuide:0x1d41a86c0'UIButtonBar.flexibleSpaceEqualSizeLayoutGuide'.minX{id: 4582}, UILayoutGuide:0x1d41a86c0'UIButtonBar.flexibleSpaceEqualSizeLayoutGuide'.minY{id: 4583}, UILayoutGuide:0x1d41a86c0'UIButtonBar.flexibleSpaceEqualSizeLayoutGuide'.Width{id: 4506}, UILayoutGuide:0x1d41a86c0'UIButtonBar.flexibleSpaceEqualSizeLayoutGuide'.Height{id: 4584}
|   |   |   |   |   *<UILayoutGuide: 0x1d41a87a0 - "UIButtonBar.minimumInterItemSpaceLayoutGuide", layoutFrame = {{-16, 0}, {8, 0}}, owningView = <_UIButtonBarStackView: 0x113b5df90; frame = (16 0; 44 44); alpha = 0; layer = <CALayer: 0x1d422e560>>>- AMBIGUOUS LAYOUT for UILayoutGuide:0x1d41a87a0'UIButtonBar.minimumInterItemSpaceLayoutGuide'.minX{id: 4585}, UILayoutGuide:0x1d41a87a0'UIButtonBar.minimumInterItemSpaceLayoutGuide'.minY{id: 4586}, UILayoutGuide:0x1d41a87a0'UIButtonBar.minimumInterItemSpaceLayoutGuide'.Height{id: 4587}
|   |   |   |   |   *<UILayoutGuide: 0x1d41a6040 - "UIButtonBar.minimumInterGroupSpaceLayoutGuide", layoutFrame = {{-16, 0}, {8, 0}}, owningView = <_UIButtonBarStackView: 0x113b5df90; frame = (16 0; 44 44); alpha = 0; layer = <CALayer: 0x1d422e560>>>- AMBIGUOUS LAYOUT for UILayoutGuide:0x1d41a6040'UIButtonBar.minimumInterGroupSpaceLayoutGuide'.minX{id: 4588}, UILayoutGuide:0x1d41a6040'UIButtonBar.minimumInterGroupSpaceLayoutGuide'.minY{id: 4589}, UILayoutGuide:0x1d41a6040'UIButtonBar.minimumInterGroupSpaceLayoutGuide'.Height{id: 4590}
|   |   |   |   |   *<UILayoutGuide: 0x1d41a85e0 - "UIViewLayoutMarginsGuide", layoutFrame = {{0, 0}, {44, 44}}, owningView = <_UIButtonBarStackView: 0x113b5df90; frame = (16 0; 44 44); alpha = 0; layer = <CALayer: 0x1d422e560>>>
|   |   |   |   |   *_UITAMICAdaptorView:0x113bba900
|   |   |   |   |   |   UIButton:0x10940abb0' 返回'
|   |   |   |   |   |   |   UIImageView:0x113bbc9a0
|   |   |   |   |   |   |   UIButtonLabel:0x113bbcd00' 返回'
|   |   |   |   +UIView:0x1092be070
|   |   |   |   |   *UILabel:0x113bba2e0'添加好友'
|   |   |   |   *_UIButtonBarStackView:0x1189125f0
|   |   |   |   |   *<UILayoutGuide: 0x1d09bc2a0 - "UIButtonBar.flexibleSpaceEqualSizeLayoutGuide", layoutFrame = {{-342, 0}, {8, 0}}, owningView = <_UIButtonBarStackView: 0x1189125f0; frame = (342 0; 17 44); layer = <CALayer: 0x1d0622460>>>- AMBIGUOUS LAYOUT for UILayoutGuide:0x1d09bc2a0'UIButtonBar.flexibleSpaceEqualSizeLayoutGuide'.minX{id: 1431}, UILayoutGuide:0x1d09bc2a0'UIButtonBar.flexibleSpaceEqualSizeLayoutGuide'.minY{id: 1432}, UILayoutGuide:0x1d09bc2a0'UIButtonBar.flexibleSpaceEqualSizeLayoutGuide'.Width{id: 784}, UILayoutGuide:0x1d09bc2a0'UIButtonBar.flexibleSpaceEqualSizeLayoutGuide'.Height{id: 1433}
|   |   |   |   |   *<UILayoutGuide: 0x1d09bf560 - "UIButtonBar.minimumInterItemSpaceLayoutGuide", layoutFrame = {{-342, 0}, {8, 0}}, owningView = <_UIButtonBarStackView: 0x1189125f0; frame = (342 0; 17 44); layer = <CALayer: 0x1d0622460>>>- AMBIGUOUS LAYOUT for UILayoutGuide:0x1d09bf560'UIButtonBar.minimumInterItemSpaceLayoutGuide'.minX{id: 1434}, UILayoutGuide:0x1d09bf560'UIButtonBar.minimumInterItemSpaceLayoutGuide'.minY{id: 1435}, UILayoutGuide:0x1d09bf560'UIButtonBar.minimumInterItemSpaceLayoutGuide'.Height{id: 1436}
|   |   |   |   |   *<UILayoutGuide: 0x1d09be4c0 - "UIButtonBar.minimumInterGroupSpaceLayoutGuide", layoutFrame = {{-342, 0}, {8, 0}}, owningView = <_UIButtonBarStackView: 0x1189125f0; frame = (342 0; 17 44); layer = <CALayer: 0x1d0622460>>>- AMBIGUOUS LAYOUT for UILayoutGuide:0x1d09be4c0'UIButtonBar.minimumInterGroupSpaceLayoutGuide'.minX{id: 1437}, UILayoutGuide:0x1d09be4c0'UIButtonBar.minimumInterGroupSpaceLayoutGuide'.minY{id: 1438}, UILayoutGuide:0x1d09be4c0'UIButtonBar.minimumInterGroupSpaceLayoutGuide'.Height{id: 1439}
|   |   |   |   |   *<UILayoutGuide: 0x1d09bc1c0 - "UIViewLayoutMarginsGuide", layoutFrame = {{0, 0}, {17, 44}}, owningView = <_UIButtonBarStackView: 0x1189125f0; frame = (342 0; 17 44); layer = <CALayer: 0x1d0622460>>>
|   |   |   |   |   *_UITAMICAdaptorView:0x11892c6f0
|   |   |   |   |   |   UIButton:0x10860c7c0
|   |   |   |   |   |   |   UIImageView:0x118905300
|   |   |   |   *UILabel:0x1092d6100'朋友'
|   |   |   +_UINavigationBarModernPromptView:0x118910a10
|   |   |   |   *UILabel:0x118910c10

Legend:
	* - is laid out with auto layout
	+ - is laid out manually, but is represented in the layout engine because translatesAutoresizingMaskIntoConstraints = YES
	• - layout engine host
```
