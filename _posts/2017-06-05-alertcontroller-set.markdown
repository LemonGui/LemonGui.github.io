---
layout:     post
title:      "UIAlertController的Alert设置"
subtitle:   "开发笔记"
date:       2017-06-05
author:     "Lemon"
header-img: "img/about-bg.jpg"
tags:
    - iOS开发
    - UIAlertController
---

>UIAlertController的Alert设置。

###1. UIAlertController弹窗设置：

```
/** 色值 RGB **/
#define RGB_A(r, g, b, a) [UIColor colorWithRed:(CGFloat)(r)/255.0f green:(CGFloat)(g)/255.0f blue:(CGFloat)(b)/255.0f alpha:(CGFloat)(a)]
#define RGB(r, g, b) RGB_A(r, g, b, 1)
#define RGB_HEX(__h__) RGB((__h__ >> 16) & 0xFF, (__h__ >> 8) & 0xFF, __h__ & 0xFF)
```

```
 UIAlertController * alertController = [UIAlertController alertControllerWithTitle:@"屏蔽说明" message:@"" preferredStyle:UIAlertControllerStyleAlert];
 
 NSMutableAttributedString *alertControllerMessageStr = [[NSMutableAttributedString alloc] initWithString:@"1.不看他的帖子\n股吧中不再显示他的帖子\n2.禁止他与我互动\n他无法对我转发、评论、@、赞\n3.禁止他关注我\n解除关注关系并禁止他关注我"];
                    [alertControllerMessageStr addAttributes:@{NSForegroundColorAttributeName:[UIColor grayColor]} range:NSMakeRange(8,12)];
                    [alertControllerMessageStr addAttributes:@{NSForegroundColorAttributeName:[UIColor grayColor]} range:NSMakeRange(30, 15)];
                    [alertControllerMessageStr addAttributes:@{NSForegroundColorAttributeName:[UIColor grayColor]} range:NSMakeRange(54, 14)];
                    [alertController setValue:alertControllerMessageStr forKey:@"attributedMessage"];
                    
                    //左对齐
                    UIView *messageParentView = [__weakMe getMessageParentView:alertController.view];
                    if (messageParentView && messageParentView.subviews.count > 1) {
                        UILabel *messageLb = messageParentView.subviews[1];
                        messageLb.textAlignment = NSTextAlignmentLeft;
                    }
                    
                    UIAlertAction *cancelAction = alertController.actions.firstObject;
                    [cancelAction setValue:[UIColor grayColor] forKey:@"titleTextColor"];
                    UIAlertAction *okAction = alertController.actions.lastObject;
                    [okAction setValue:RGB_HEX(0xea5504) forKey:@"titleTextColor"]; 
```

```
- (UIView *)getMessageParentView:(UIView *)view {
    for (UIView *subView in view.subviews) {
        if ([subView isKindOfClass:[UILabel class]]) {
            return view;
        }else{
            UIView *resultV = [self getMessageParentView:subView];
            if (resultV) return resultV;
        }
    }
    return nil;
}
```

###2.效果图：
![](https://raw.githubusercontent.com/LemonGui/LemonGui.github.io/master/img/otherImage/alert_set.jpeg)


