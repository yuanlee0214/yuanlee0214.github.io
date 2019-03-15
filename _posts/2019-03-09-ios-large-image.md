---
title: "iOS加载大图避免占用过多内存"
layout: post
date: 2019-3-9 12:00
image: #/assets/images/markdown.jpg
headerImage: false
tag:
- iOS
category: blog
author: #Yuan Lee
description: #Markdown summary with different options
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="http://localhost:4000/assets/images/profile.jpg" height="20" width="20" align="absmiddle">'
---

## Summary

在需要加载大图的时候，创建 imageView 继承自此 LargeImageView 即可

### 前言

#### 首先分析一下图片加载到内存的大小的计算方法

- 如果是位图，则位图多大，加载到内存中所占用的空间就是多大

- 如果是非位图的图片，比如jpg/png,则需要解码成位图，解码出来的位图多大也就是意味着该jpg/png占用内存多大。

- 位图的大小计算公式：图片 width x heigth x 4（ARGB）

- 比如一张 1000 x 1000 的png图片，则其解压出来的位图的占用大小为1000 x 1000 x 4(即3.81MB左右)，也就是说这张图片会占用3.81MB左右的内存。

#### Apple官方的解决办法

- 把大图片进行分片加载。根据原图的分片CGRect获取到原图的一个小分片p1，根据缩放后的分片CGRect和 p1生成缩放后的小分片，绘制到屏幕，这样一直循环把原图片全部的每个小分片全部生成对应缩放的小分片，然后绘制。

---


<h4>在需要加载大图的时候，创建 imageView 继承自此 LargeImageView 即可</h4>
{% highlight raw %}
LargeImageView *imageView = [LargeImageView new];
[imageView setImage:image];
{% endhighlight %}
<h5>头文件 "LargeImageView.h"</h5>
{% highlight raw %}
//
//  LargeImageView.h
//
//  Created by YuanLee on 2019/3/9.
//  Copyright © 2019 YuanLee. All rights reserved.
//

#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface LargeImageView : UIView
- (void)setImage:(UIImage *)image;
@end

NS_ASSUME_NONNULL_END
{% endhighlight %}

<h5>实现文件 "LargeImageView.m"</h5>
{% highlight raw %}
//
//  LargeImageView.m
//
//  Created by YuanLee on 2019/3/9.
//  Copyright © 2019 YuanLee. All rights reserved.
//

#import "LargeImageView.h"

@interface LargeImageView ()
{
    long long tiledCount;
    UIImage *originImage;
    CGRect imageRect;
    CGFloat imageScale_w;
    CGFloat imageScale_h;
}
@end

@implementation LargeImageView

- (void)setImage:(UIImage *)image {
    if (tiledCount == 0) {
        tiledCount = 81;
    }
    originImage = image;
    [self setBackgroundColor:[UIColor whiteColor]];
    imageRect = CGRectMake(0.0f,
                           0.0f,
                           CGImageGetWidth(originImage.CGImage),
                           CGImageGetHeight(originImage.CGImage));
    imageScale_w = self.frame.size.width/imageRect.size.width;
    imageScale_h = self.frame.size.height/imageRect.size.height;
    CATiledLayer *tiledLayer = (CATiledLayer *)[self layer];
    
    int scale = (int)MAX(1/imageScale_w, 1/imageScale_h);
    
    int lev = ceil(scale);
    tiledLayer.levelsOfDetail = 1;
    tiledLayer.levelsOfDetailBias = lev;
    
    if (tiledCount > 0){
        NSInteger tileSizeScale = sqrt(tiledCount)/2;
        CGSize tileSize = self.bounds.size;
        tileSize.width /=tileSizeScale;
        tileSize.height /=tileSizeScale;
        tiledLayer.tileSize = tileSize;
    }
}

+ (Class)layerClass {
    return [CATiledLayer class];
}

- (void)drawRect:(CGRect)rect {
    @autoreleasepool {
        CGRect imageCutRect = CGRectMake(rect.origin.x / imageScale_w,
                                         rect.origin.y / imageScale_h,
                                         rect.size.width / imageScale_w,
                                         rect.size.height / imageScale_h);
        CGImageRef imageRef = CGImageCreateWithImageInRect(originImage.CGImage, imageCutRect);
        UIImage *tileImage = [UIImage imageWithCGImage:imageRef];
        CGContextRef context = UIGraphicsGetCurrentContext();
        UIGraphicsPushContext(context);
        [tileImage drawInRect:rect];
        CGImageRelease(imageRef);
        UIGraphicsPopContext();
    }
}

@end
{% endhighlight %}