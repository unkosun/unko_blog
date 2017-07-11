---
title: iOS简单无埋点统计的实现（一）——数据收集篇
date: 2017-07-10 15:06:40
tags:	
	- iOS
	- 无埋点
	- 统计
---
无埋点统计使用的最关键技术就是 Method Swizzling。本篇内容将介绍如何收集数据。

目前我们主要收集三类数据：
1. 全局数据：包含 App信息和设备信息
2. 页面浏览数据：统计页面访问时长信息
3. 点击数据：统计点击事件数据信息

## 一、全局数据
在 App启动初始化 统计模块的时候，就会生成一条关于 App信息的记录。统计 App的设备，系统版本，还有 App的版本等。每次启动 App,有且仅有1条全局数据记录。

## 二、页面浏览事件
分别 Hook `UIViewController` 的 `viewWillAppear:` 和 `viewWillDisappear:` 函数，用来统计页面显示的时间戳和裂开的时间戳。

## 三、点击事件
### Hook 关键点
* 基于控件的点击事件
  `UIControl` 的 `addTarget:action:forControlEvents:`

* 基于手势的点击事件
  `UIGestureRecognizer` 的 `initWithTarget:action:` 和 `addTarget:action:`  

添加action用于无埋点的统计

*  `UITableView` 和 `UICollectionView` 的点击事件


### 如何定位点击 view 的 ？

* Response Chain
  UIWindow/../../TestViewController.view/Button
* 同类型子 view 下标引用
  UIWindow/../../TestViewController.view/Button1
* 设置唯一标识

```
@interface UIView (AutoAnalytics)
@property (nonatomic, strong, nullable) NSString * autoAnalyticsUniqueID; //别名
@end
```


##  四、数据定义

```
@interface AutoAnalyticsLog : BaseEntity
@property (nonatomic, strong) NSString * sessionID;         ///< session id
@property (nonatomic, strong) NSString * name;              ///< 名称；
@property (nonatomic, assign) NSTimeInterval dateInterval;  ///< 时间戳
@property (nonatomic, strong) NSString * parentID;          ///< 父统计节点的ID
@property (nonatomic, strong) NSDictionary * logInfo;
@end
```


### 记录关联
全局数据记录每此 App启动只生成1条，而浏览记录和点击记录是可以多条的。这个时候我们就需要用到记录和记录之前的关联。
一条点击事件的记录如何来确定是在哪个页面浏览时发生的？一条浏览事件的记录如何来确定是哪次 App启动产生的？

答案是 通过parentID关联 sessionID。

## 五、总结
1. 因为没有相关无埋点统计资料，很多时候需要自己摸索。
2. 参考同类产品：view的路径，参考了GrowingIO 上次来公司分享中提到的 path。关于手势点击部分的关键点 Hook,通过调试参考了听云。
3. Hook关键点：在考虑 view点击事件的 Hook时很容易考虑不全。



