---
layout:     post
title:      "自定义导航栏后侧滑返回功能失效"
subtitle:   ""
date:       2016-06-06
author:     "Lemon"
header-img: "img/post-bg-2016.jpg"
tags:
    - iOS开发
---


>从iOS7开始，系统为UINavigationController提供了一个interactivePopGestureRecognizer用于右滑返回(pop),但是，如果自定了当前视图控制器leftBarButtonItem，该手势就失效了。

解决方法：自定义`UINavigationController`，实现其代理方法:
```
- (void)navigationController:(UINavigationController *)navigationController didShowViewController:(UIViewController *)viewController animated:(BOOL)animated;

```
如果push的是非根控制器，设置`self.interactivePopGestureRecognizer.delegate = nil;`。
如果是根控制器，将原来的代理重新设置即可。
具体代码如下：
```
#import "LGNavigationController.h"

@interface LGNavigationController ()<UINavigationControllerDelegate>

@property (nonatomic,strong) id popDelegate;

@end

- (void)viewDidLoad {
    [super viewDidLoad];
    //代理    
    self.popDelegate = self.interactivePopGestureRecognizer.delegate;
    self.delegate = self;
}

#pragma UINavigationControllerDelegate方法
- (void)navigationController:(UINavigationController *)navigationController didShowViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    //实现滑动返回功能
    //清空滑动返回手势的代理就能实现
     self.interactivePopGestureRecognizer.delegate =  viewController == self.viewControllers[0]? self.popDelegate : nil;
}


```
