---
layout:     post
title:      "图片压缩上传"
subtitle:   "开发笔记"
date:       2016-11-18
author:     "Lemon"
header-img: "img/about-bg.jpg"
tags:
    - iOS开发
    - 图片压缩
---


前段时间处理了下图片上传压缩的问题，在这里记录下。iPhone拍摄的照片2-3M可压缩至30-60KB左右，清晰度还可以接受,如果一次处理多张高清图，放入子线程中进行压缩
```
/**
 *  图片上传压缩
 *  @param source_image    原图片
 *  @param compressQuality 压缩系数 0-1
 *  默认参考大小30kb,一般用该方法可达到要求，压缩系数可根据压缩后的清晰度权衡，项目里我用的0.2😆
 */
+ (NSData *)resetSizeOfImageData:(UIImage *)source_image compressQuality:(CGFloat)compressQuality
{
    return  [self resetSizeOfImageData:source_image referenceSize:30 compressQuality:compressQuality];
}
```


```
/**
 *  图片上传压缩
 *  @param source_image    原图片
 *  @param referenceSize   上传的参考大小**KB
 *  @param compressQuality 压缩系数 0-1
 *  @return                imageData
 */
+ (NSData *)resetSizeOfImageData:(UIImage *)source_image referenceSize:(NSInteger)maxSize compressQuality:(CGFloat)compressQuality
{
    //先调整分辨率
    CGSize newSize = CGSizeMake(source_image.size.width, source_image.size.height);
    NSInteger tempHeight = newSize.height / 1024;
    NSInteger tempWidth = newSize.width / 1024;
    
    if (tempWidth > 1.0 && tempWidth > tempHeight) {
        newSize = CGSizeMake(source_image.size.width / tempWidth, source_image.size.height / tempWidth);
    }
    else if (tempHeight > 1.0 && tempWidth < tempHeight){
        newSize = CGSizeMake(source_image.size.width / tempHeight, source_image.size.height / tempHeight);
    }
    UIGraphicsBeginImageContext(newSize);
    [source_image drawInRect:CGRectMake(0,0,newSize.width,newSize.height)];
    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    //调整大小
    NSData *imageData = UIImageJPEGRepresentation(newImage,1.0);
    NSUInteger sizeOrigin = [imageData length];
    NSUInteger sizeOriginKB = sizeOrigin / 1024;
    if (sizeOriginKB > maxSize) {
        NSData *finallImageData = UIImageJPEGRepresentation(newImage,compressQuality);
        return finallImageData;
    }
    return imageData;
}
```