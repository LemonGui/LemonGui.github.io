---
layout:     post
title:      "PhotoKit的使用以及简易相片选择器的创建"
subtitle:   "开发笔记"
date:       2016-11-21
author:     "Lemon"
header-img: "img/about-bg.jpg"
tags:
    - iOS开发
    - PhotoKit
---

>随着iOS10的推出，Xcode 8 最低支持iOS 8.0，PhotoKit可以完全替代ALAssetsLibrary来管理相册资源。本文主要介绍PhotoKit的基本使用,做一个简易的相片选择器。

#####一、基本概念：
**PHAsseet**:代表照片库中的一个资源，跟 ALAsset 类似，通过 PHAsset 可以获取和保存资源;
**PHFetchOptions**:获取资源时的参数，可以传nil，即使用系统默认值；
**PHFetchResult**:表示一系列的资源集合，也可以是相册的集合；
**PHAssetCollection**:表示一个相册或者一个时刻，或者是一个「智能相册(系统提供的特定的一系列相册，例如：最近删除，视频列表，收藏等等);
**PHCollectionList**:表示一组PHCollection,而它本身也是一个PHCollection,因此PHCollection作为一个集合，可以包含其他集合，例如：照片 - 时刻 - 精选 - 年度；
**PHImageManager**:用于处理资源的加载，加载图片的过程带有缓存处理，可以通过传入一个 PHImageRequestOptions 控制资源的输出尺寸等规格;
**PHImageRequestOptions**:如上面所说，控制加载图片时的一系列参数。

![](http://upload-images.jianshu.io/upload_images/2203501-b6405dd63ab659d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####二、PhotoKit的基本使用：
######为便于使用，封装一个单例类PhotoCatcherManager，实现以下方法：
1.获取全部相册：

```
-(NSMutableArray<PHAssetCollection*> *)getPhotoListDatas{
    NSMutableArray *dataArray = [NSMutableArray array];
    //获取资源时的参数，为nil时则是使用系统默认值
    PHFetchOptions *fetchOptions = [[PHFetchOptions alloc]init];
    //列出所有的智能相册
    PHFetchResult *smartAlbumsFetchResult = [PHAssetCollection fetchAssetCollectionsWithType:PHAssetCollectionTypeSmartAlbum subtype:PHAssetCollectionSubtypeSmartAlbumUserLibrary options:fetchOptions];
    [dataArray addObject:[smartAlbumsFetchResult objectAtIndex:0]];
    //列出所有用户创建的相册
    PHFetchResult *smartAlbumsFetchResult1 = [PHAssetCollection fetchTopLevelUserCollectionsWithOptions:fetchOptions];
    for (PHAssetCollection *sub in smartAlbumsFetchResult1) {
        [dataArray addObject:sub];
    }
    return dataArray;
}
```
2.获取一个相册的结果集(按时间排序)

```
//获取某个相册的结果集
-(PHFetchResult<PHAsset *> *)getFetchResult:(PHAssetCollection *)assetCollection ascend:(BOOL)ascend{
    PHFetchOptions *options = [[PHFetchOptions alloc] init];
    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 9.0f) {
        options.includeAssetSourceTypes = PHAssetSourceTypeUserLibrary;
    }
    //时间排序
    options.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"creationDate" ascending:ascend]];
    PHFetchResult *allPhotos = [PHAsset fetchAssetsInAssetCollection:assetCollection options:nil];
    return allPhotos;
}
```
3.获取某个类型的结果集(按时间排序)

```
+ (PHFetchResult<PHAsset *> *)getFetchResultWithMediaType:(PHAssetMediaType)mediaType ascend:(BOOL)ascend{
    PHFetchOptions *options = [[PHFetchOptions alloc] init];
    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 9.0f) {
        options.includeAssetSourceTypes = PHAssetSourceTypeUserLibrary;
    }
    options.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"creationDate" ascending:ascend]];
     return  [PHAsset fetchAssetsWithMediaType:mediaType options:options];
}
```
---
4.获取系统相册CameraRoll 的结果集(按时间排序)

```
-(PHFetchResult<PHAsset *> *)getCameraRollFetchResulWithAscend:(BOOL)ascend{
    //获取系统相册CameraRoll 的结果集
    PHFetchOptions *fetchOptions = [[PHFetchOptions alloc]init];
    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 9.0f) {
        fetchOptions.includeAssetSourceTypes = PHAssetSourceTypeUserLibrary;
    }
    fetchOptions.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"creationDate" ascending:ascend]];
    PHFetchResult *smartAlbumsFetchResult = [PHAssetCollection fetchAssetCollectionsWithType:PHAssetCollectionTypeSmartAlbum subtype:PHAssetCollectionSubtypeSmartAlbumUserLibrary options:nil];
    PHFetchResult *fetch = [PHAsset fetchAssetsInAssetCollection:[smartAlbumsFetchResult objectAtIndex:0] options:fetchOptions];
    return fetch;
}
```
5.获取单张高清图,progressHandler为从iCloud下载进度

```
-(void)getImageHighQualityForAsset:(PHAsset *)asset progressHandler:(void(^)(double progress, NSError * error, BOOL *stop, NSDictionary * info))progressHandler resultHandler:(void (^)(UIImage* result, NSDictionary * info))resultHandler{
   
    CGSize imageSize = [self imageSizeForAsset:asset];
    PHImageRequestOptions * options = [[PHImageRequestOptions alloc] init];
    //设置该模式，若本地无高清图会立即返回缩略图，需要从iCloud下载高清，会再次调用resultHandler返回下载后的高清图
    options.deliveryMode = PHImageRequestOptionsDeliveryModeOpportunistic;
    options.resizeMode = PHImageRequestOptionsResizeModeFast;
    options.networkAccessAllowed = YES;
    options.progressHandler = ^(double progress, NSError *__nullable error, BOOL *stop, NSDictionary *__nullable info){
      
        if (progressHandler) {
            dispatch_async(dispatch_get_main_queue(), ^{
                progressHandler(progress,error,stop,info);
            });
        }
    };
    
    [[PHCachingImageManager defaultManager] requestImageForAsset:asset targetSize:imageSize contentMode:PHImageContentModeAspectFill options:options resultHandler:^(UIImage * _Nullable result, NSDictionary * _Nullable info) {
        //判断高清图
        //   BOOL downloadFinined = ![[info objectForKey:PHImageCancelledKey] boolValue] && ![info objectForKey:PHImageErrorKey] && ![[info objectForKey:PHImageResultIsDegradedKey] boolValue];
        if (result && resultHandler) {
            resultHandler(result,info);
        }
    }];
}
```
私有方法，根据屏幕获取图片的大小

```
-(CGSize)imageSizeForAsset:(PHAsset *)asset{
    CGFloat photoWidth = [UIScreen mainScreen].bounds.size.width;
    CGFloat multiple = [UIScreen mainScreen].scale;
    CGFloat aspectRatio = asset.pixelWidth / (CGFloat)asset.pixelHeight;
    CGFloat pixelWidth = photoWidth * multiple;
    CGFloat pixelHeight = pixelWidth / aspectRatio;
    return  CGSizeMake(pixelWidth, pixelHeight);
}
```
6.获取单张缩略图

```
-(void)getImageLowQualityForAsset:(PHAsset *)asset targetSize:(CGSize)targetSize resultHandler:(void (^)(UIImage* result, NSDictionary * info))resultHandler{
    [[PHCachingImageManager defaultManager] requestImageForAsset:asset targetSize:targetSize contentMode:PHImageContentModeAspectFill options:nil resultHandler:^(UIImage * _Nullable result, NSDictionary * _Nullable info) {
        if (result && resultHandler) {
            resultHandler(result,info);
        }
    }];
}
```
7.同时获取多张图片(高清)，全部为高清图resultHandler才执行，需要从iCloud下载时progressHandler提供每张进度（P.S:内部压缩图片方法[实现](https://lemongui.github.io/2016/11/18/image-upload/)）

```
-(void)getImagesForAssets:(NSArray<PHAsset *> *)assets progressHandler:(void(^)(double progress, NSError * error, BOOL *stop, NSDictionary * info))progressHandler resultHandler:(void (^)(NSArray<NSDictionary *> *))resultHandler{
    NSMutableArray * callBackPhotos = [NSMutableArray array];

    //此处在子线程中执行requestImageForAsset原因：options.synchronous设为同步时,options.progressHandler获取主队列会死锁
    NSOperationQueue * queue = [[NSOperationQueue alloc] init];
    queue.maxConcurrentOperationCount = 1;
    
    for (PHAsset * asset in assets) {
        CGSize imageSize = [self imageSizeForAsset:asset];
        PHImageRequestOptions * options = [[PHImageRequestOptions alloc] init];
//        options.deliveryMode = PHImageRequestOptionsDeliveryModeFastFormat;
        options.resizeMode = PHImageRequestOptionsResizeModeExact;
        options.networkAccessAllowed = YES;
        //同步保证取出图片顺序和选择的相同，deliveryMode默认为PHImageRequestOptionsDeliveryModeHighQualityFormat
        options.synchronous = YES;
        
        options.progressHandler = ^(double progress, NSError *__nullable error, BOOL *stop, NSDictionary *__nullable info){
            dispatch_async(dispatch_get_main_queue(), ^{
                progressHandler(progress,error,stop,info);
            });
        };
        
        NSBlockOperation * op = [NSBlockOperation blockOperationWithBlock:^{
            [[PHCachingImageManager defaultManager] requestImageForAsset:asset targetSize:imageSize contentMode:PHImageContentModeAspectFill options:options resultHandler:^(UIImage * _Nullable result, NSDictionary * _Nullable info) {
                //resultHandler默认在主线程，requestImageForAsset在子线程执行后resultHandler变为在子线程
                if (result) {
                    //压缩图片，可用于上传
                    NSData * data = [UIImage resetSizeOfImageData:result compressQuality:0.2];
                    UIImage * image = [UIImage imageWithData:data];
                    NSDictionary * dic = @{EEPhotoImage:image,EEPhotoName:asset.localIdentifier};
                    [callBackPhotos addObject:dic];
                    if (resultHandler && callBackPhotos.count == assets.count) {
                        dispatch_async(dispatch_get_main_queue(), ^{
                            resultHandler(callBackPhotos);
                        });
                    }
                }
            }];
        }];
        [queue addOperation:op];
    }
}

```
8.获取相册授权状态

```
+(void)requestAuthorizationHandler:(void(^)(BOOL isAuthorized))handler {
    
    if (handler) {
        [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status) {
            dispatch_async(dispatch_get_main_queue(), ^{
                if (status == PHAuthorizationStatusAuthorized) {
                    handler(YES);
                }else{
                    handler(NO);
                }
            });
        }];
    }
}
```

#####三、创建相片选择器：
######1.先看效果图（注意同时拉取多张大图时内存处理）
![效果图](http://upload-images.jianshu.io/upload_images/2203501-9bf33e275349c25b.gif?imageMogr2/auto-orient/strip)
---
主要包含以下几点：

* ViewControllerA含有一个UITextView，我们从相册选取一张或多张图片让其展示。
* ViewControllerB是相片选择界面，有UICollectionView构成，展示的是相册图片缩略图。
* ViewControllerB界面点击图片可查看该图片高清图。

######2.实现
2.1. ViewControllerA.m

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.textView.textContainerInset = UIEdgeInsetsZero;
    self.textView.textContainer.lineFragmentPadding = 0;
    self.automaticallyAdjustsScrollViewInsets = NO;
     self.navigationItem.rightBarButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"相册" style:UIBarButtonItemStylePlain target:self action:@selector(enterPhotoAlbum)];
}

-(void)enterPhotoAlbum{
    WS(__weakMe);
    [PhotoCatcherManager requestAuthorizationHandler:^(BOOL isAuthorized) {
        if (isAuthorized) {
            
            PhotoController * vc = [PhotoController new];
            vc.photoCallBackBlock = ^(NSArray * photos){
                [__weakMe setTextViewWithPhotos:photos];
            };
            [__weakMe.navigationController pushViewController:vc animated:YES];

        }else{
            UIAlertController * vc = [UIAlertController alertControllerWithTitle:@"您没赋予本程序相机权限" message:@"您可以在iOS系统的“设置→隐私→相机”中赋予本程序相机权限" preferredStyle:UIAlertControllerStyleAlert];
            UIAlertAction * action = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
            }];
            [vc addAction:action];
            [__weakMe presentViewController:vc animated:NO completion:NULL];
            return ;
        }
    }];
}

-(void)setTextViewWithPhotos:(NSArray*)photos{
    for (NSDictionary * dic in photos) {
        UIImage * image = dic[EEPhotoImage];
        NSUInteger location = self.textView.selectedRange.location ;
        NSTextAttachment * attachMent = [[NSTextAttachment alloc] init];
        attachMent.image = image;
        CGSize size = [self displaySizeWithImage:image];
        attachMent.bounds = CGRectMake(0, 0, size.width,size.height);
        NSAttributedString * attStr = [NSAttributedString attributedStringWithAttachment:attachMent];
        NSMutableAttributedString *textViewString = [self.textView.attributedText mutableCopy];
        [textViewString insertAttributedString:attStr atIndex:location];
        
        [textViewString addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:19] range:NSMakeRange(0, textViewString.length)];
        self.textView.attributedText = textViewString;
        self.textView.selectedRange = NSMakeRange(location + 1,0);
    }
}

//显示图片的大小 （全屏）
- (CGSize)displaySizeWithImage:(UIImage *)image {
    CGSize displaySize;
    if (image.size.width !=0 ) {
        CGFloat _widthRadio = APPCONFIG_UI_SCREEN_FWIDTH / image.size.width;
        CGFloat _imageHeight = image.size.height * _widthRadio;
        displaySize = CGSizeMake(APPCONFIG_UI_SCREEN_FWIDTH, _imageHeight);
    }else{
        displaySize = CGSizeZero;
    }
    return displaySize;
}
```
2.2. ViewControllerB.h  ( PhotoController )

```
@interface PhotoController : UIViewController
@property (nonatomic,copy) void(^(photoCallBackBlock))(NSArray *);
@end
```

2.3.ViewControllerB.m 部分代码( PhotoController )

```
- (void)viewDidLoad {
    self.manager = [PhotoCatcherManager sharedInstance];
    self.allPhotos = [PhotoCatcherManager getFetchResultWithMediaType:PHAssetMediaTypeImage ascend:NO];
}
#pragma mark- UICollectionViewDataSource
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return _allPhotos.count;
}

-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    PhotoCell * cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"cell" forIndexPath:indexPath];
    cell.selectedBtn.selected = NO;
    for (NSIndexPath * index in _selectedArray) {
        if (index.row == indexPath.row) {
            cell.selectedBtn.selected = YES;
        }
    }
    
    PHAsset *asset = self.allPhotos[indexPath.item];
    
    WS(weakSelf);
    [cell setBtnClickBlock:^(UIButton * btn) {
        if (btn.selected) {
            if (weakSelf.selectedArray.count>=20) {
              MBProgressHUD * hud = [MBProgressHUD showHUDAddedTo:[UIApplication sharedApplication].keyWindow animated:YES];
                hud.mode = MBProgressHUDModeText;
                hud.label.text = @"本次最多选择20张照片";
                [hud hideAnimated:YES afterDelay:2];
                btn.selected = NO;
                return ;
            }
            [weakSelf.selectedArray addObject:indexPath];
        }else{
            [weakSelf.selectedArray removeObject:indexPath];
        }
    }];
    
#pragma mark 单张缩略图
    [_manager getImageLowQualityForAsset:asset targetSize:ImageSize resultHandler:^(UIImage *result, NSDictionary *info) {
        if (result) {
            cell.image = result;
        }
    }];
    
    return cell;
}

#pragma mark- 单张图片
-(void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath{
    
    PHAsset *asset = self.allPhotos[indexPath.item];
    WS(weakSelf);
    [_manager getImageHighQualityForAsset:asset progressHandler:^(double progress, NSError *error, BOOL *stop, NSDictionary *info) {
        if (error) {
            NSLog(@"iClound error:  %@ ",error);
            return ;
        }
        _hud.hidden = NO;
        _hud.progress = progress;
        _hud.label.text = @"同步iCloud中";
        NSLog(@"downloading the image from iCloud, progress:~~ %f ~~%@",progress,info);
        
    }  resultHandler:^(UIImage *result, NSDictionary *info) {
        BOOL downloadFinined = ![[info objectForKey:PHImageResultIsDegradedKey] boolValue];
        if (downloadFinined) {
            [weakSelf.hud setHidden:YES];
        }
        weakSelf.browserView.image = result;
        weakSelf.browserView.hidden = NO;
    }];
    
}

#pragma mark- 选取多张图片
-(void)finishSelected{
    NSMutableArray * assets = [NSMutableArray array];
    WS(weakSelf);
    for (NSIndexPath * indexPath in _selectedArray) {
        PHAsset * asset = self.allPhotos[indexPath.item];
        [assets addObject:asset];
    }

    [_manager getImagesForAssets:assets progressHandler:^(double progress, NSError *error, BOOL *stop, NSDictionary *info) {
        if (error) {
            NSLog(@"iClound error:  %@ ",error);
            return ;
        }
                weakSelf.hud.hidden = NO;
                weakSelf.hud.progress = progress;
                weakSelf.hud.label.text = @"同步iCloud中";
        NSLog(@"downloading the image from iCloud, progress:~~ %f ~~%@",progress,info);
        
    } resultHandler:^(NSArray<NSDictionary *> *result) {
        weakSelf.hud.hidden = YES;
        [weakSelf.navigationController popViewControllerAnimated:YES];
        weakSelf.photoCallBackBlock(result);
    }];
}
```

