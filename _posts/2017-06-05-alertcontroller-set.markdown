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

1. UIAlertController弹窗设置：

```
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


