+++
title = "Debug CUICatalog Invalid Asset Name Supplied Null"
date = 2015-10-29T16:30:59+08:00
tags = ["iOS", "Xcode"]

+++

### 起因

今天看到了一个Xcode Log出了一个错误 `` CUICatalog:Invalid asset name supplied: (null) ``, Google了一下可能是``+[UIImage imageNamed:]``调用的时候, name为nil. 虽然在运行的时候界面一切正常, 但是看到这个log还是想干掉它.

### 方法

需要解决的问题是查找所有``+[UIImage imageNamed:]``调用的时候, 找到name是nil的地方, 但是整个项目一搜 "imageNamed" 显示 "267 results in 117 files", 人工查找就算了吧.

一开始想到的是用Method Swizzle来修改``+[UIImage imageNamed:]``的实现, 在name为nil的时候用断言, 查看调用栈. 但是想想写了debug之后还得删掉, 比较麻烦.

于是机智的我想到了用``Symbolic Breakpoint``.

### 解决方案

1. 在Xcode的Breakpoint Navigator点击加号, 选择 ``Add Symbolic Breakpoint ``.
2. 右键选择Breakpoint选择 ``Edit Breakpoint`` , 在Symbol填入``+[UIImage imageNamed:]`` , 在Condition填入 ``[(NSString *)$arg3 length] == 0`` 或者 ``$arg3 == nil``. 可以自己尝试``po $arg1``,  ``po $arg2``试试看.
![Debug1](/img/Debug1.png)

3. 运行程序, 直到程序进入断点. 打开Debug Navigator观察调用栈, 最顶部的一定是``+[UIImage imageNamed:]`` , 点击调用栈下一条, 能够看到有调用到imageNamed的代码, 就是name为nil的地方.
![Debug2](/img/Debug2.png)
4. Fix it.

### 其他

会发现这个问题, 是由于今天看到了一个很好玩的函数 ``instrumentObjcMessageSends()``. 它是用来监控Mac/iOS模拟器中Runtime message的方法. 

```ObjC
#import <objc/runtime.h>
//Declare
void instrumentObjcMessageSends();

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  //Set NO to disable
  instrumentObjcMessageSends(YES);
}

```
具体用法可以自己Google.
