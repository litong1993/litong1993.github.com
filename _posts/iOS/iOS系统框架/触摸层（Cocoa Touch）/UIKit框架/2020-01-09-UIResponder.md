---
layout: post
title: 'UIResponder'
date: 2020-01-09
author: 李童
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: iOS

---

## UIResponder概述

App与用户进行交互，基本上是依赖于各种各样的事件。例如，用户点击界面上的按钮，我们需要触发一个按钮点击事件，并进行相应的处理，以给用户一个响应。比如UIView能够处理事件，其本身就是一个事件响应者，可以处理点击等事件，而这些事件就是在UIResponder类中定义的。

一个**UIResponder**类为那些需要响应并处理事件的对象定义了一组接口。这些事件主要分为三类：**触摸事件(touch events）**，**运动事件(motion events)**和**远程控制事件**。**UIResponder**类为每类事件都定义了一组接口，这个我们将在下面详细描述。

在UIKit中，**UIApplication**、**UIView**、**UIViewController**这几个类都是直接继承自**UIResponder**类。另外SpriteKit中的**SKNode**也是继承自**UIResponder**类。因此UIKit中的视图、控件、视图控制器，以及我们自定义的视图及视图控制器都有响应事件的能力。

## API解析

#### 响应链管理

```objective-c
//下一个响应者，只读属性，需要子类重写
@property(nonatomic, readonly, nullable) UIResponder *nextResponder;
```

- UIResponder类并不自动保存或设置下一个响应者，该方法的默认实现是返回nil。
- 子类的实现必须重写这个方法来设置下一响应者。
- UIView的实现是返回**管理它的UIViewController对象**(**如果它有**)或者**其父视图**。
- 而UIViewController的实现是返回它的视图的父视图；
- UIWindow的实现是返回app对象；
- 而UIApplication的实现是返回nil。

```objective-c

//判定当前对象是否是第一响应者
@property(nonatomic, readonly) BOOL isFirstResponder;
```

```objective-c
//是否可以成为第一响应者，只有返回YES才能成为第一响应者，默认返回为NO
@property(nonatomic, readonly) BOOL canBecomeFirstResponder;    // default is NO
//将当前对象设置为第一响应者，如果在子类中重写则必须调用Super实现
- (BOOL)becomeFirstResponder;
```

一个响应对象只有在***当前响应者***<u>能放弃第一响应者状态</u>(canResignFirstResponder)且<u>自身能成为第一响应者</u>(canBecomeFirstResponder)时才会成为第一响应者。需要注意的是只有当视图是视图层次结构的一部分时才调用这个方法。如果视图的**window**属性不为空时，视图才在一个视图层次结构中；如果该属性为nil，则视图不在任何层次结构中。我们不能向一个不在视图层次结构中的视图发送这个消息，其结果是未定义的。

```objective-c
//是否可以辞去第一响应者，只有返回YES才能辞去第一响应者，默认返回为NO
@property(nonatomic, readonly) BOOL canResignFirstResponder;    // default is YES
//当前对象辞去第一响应者，如果在子类中重写则必须调用Super实现
- (BOOL)resignFirstResponder;
```

canResignFirstResponder可以在需要时返回NO，例如：当前的输入不符合要求。

重写resignFirstResponder时默认返回YES，也可以返回NO来拒绝辞去第一响应者。

#### 响应触摸事件

```objective-c
// 当一个或多个手指触摸到一个视图或窗口
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
// 当与事件相关的一个或多个手指在视图或窗口上移动时
- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
// 当一个或多个手指从视图或窗口上抬起时
- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
// 当一个系统事件取消一个触摸事件时
- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
//普通的 iphone 应用开发中不会用到这个方法，这个方法是为了 Apple Pencil 的特性设计的   
- (void)touchesEstimatedPropertiesUpdated:(NSSet<UITouch *> *)touches API_AVAILABLE(ios(9.1));
```

#### 响应按压事件

```objective-c
// 深按API，一般用于遥控器
// 开始按压的时候调用
- (void)pressesBegan:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event NS_AVAILABLE_IOS(9_0);
// 按压改变的时候调用
- (void)pressesChanged:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event NS_AVAILABLE_IOS(9_0);
// 按压结束的时候调用
- (void)pressesEnded:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event NS_AVAILABLE_IOS(9_0);
// 当系统发出取消按压事件的时候调用
- (void)pressesCancelled:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event NS_AVAILABLE_IOS(9_0);
```

#### 响应运动事件

```objective-c
//运动开始，例如：摇一摇
- (void)motionBegan:(UIEventSubtype)motion withEvent:(nullable UIEvent *)event API_AVAILABLE(ios(3.0));
//运动结束
- (void)motionEnded:(UIEventSubtype)motion withEvent:(nullable UIEvent *)event API_AVAILABLE(ios(3.0));
//当一个系统事件取消一个运动事件时
- (void)motionCancelled:(UIEventSubtype)motion withEvent:(nullable UIEvent *)event API_AVAILABLE(ios(3.0));
```

**触摸事件、按压事件、运动事件默**认都是什么都不做。不过，UIKit中UIResponder的子类，尤其是UIView，这几个方法的实现都会把消息传递到响应链上。因此，为了不阻断响应链，我们的子类在重写时需要调用父类的相应方法；而不要将消息直接发送给下一响应者。

#### 响应远程控制事件

```objective-c
- (void)remoteControlReceivedWithEvent:(nullable UIEvent *)event API_AVAILABLE(ios(4.0));
```

接收响应者对象需要检查事件的子类型（UIEventSubtype）来确定命令，然后进行相应处理。不过，为了允许分发远程控制事件，我们必须调用**UIApplication**的**beginReceivingRemoteControlEvents**方法；而如果要关闭远程控制事件的分发，则调用**endReceivingRemoteControlEvents**方法。

#### 响应菜单命令

```objective-c
- (BOOL)canPerformAction:(SEL)action withSender:(id)sender
```

该方法默认返回YES，响应所有的菜单命令。子类可以重写这个方法来决定开启哪些菜单命令。

```
- (nullable id)targetForAction:(SEL)action withSender:(nullable id)sender API_AVAILABLE(ios(7.0));
```

默认的实现是调用**canPerformAction:withSender**:方法来确定当前对象是否可以调用**action**操作。如果可以，则返回对象本身，否则将请求传递到响应链上。可以通过重写，让请求不再向下传递；

## 知识点