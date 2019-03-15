---
title: "iOS常用宏定义"
layout: post
date: 2019-3-7 17:00
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

iOS开发中一些常用的宏定义

---

{% highlight raw %}
#pragma mark -- 屏幕宽高
#define LEESWidth [UIScreen mainScreen].bounds.size.width
#define LEESHeight [UIScreen mainScreen].bounds.size.height
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 颜色处理
#define LEEHsb(h, s, b) [UIColor colorWithHue:h/360.0f \
                                   saturation:s/100.0f \
                                   brightness:b/100.0f \
                                        alpha:1.0]
#define LEERgba(r, g, b, a) [UIColor colorWithRed:(r)/255.0 \
                                            green:(g)/255.0 \
                                             blue:(b)/255.0 \
                                            alpha:(a)]
#define LEEHexColor(c, a) [UIColor colorWithRed:(((c >> 16) & 0xFF) / 255.0) \
                                          green:(((c >> 8) & 0xFF) / 255.0) \
                                           blue:(((c) & 0xFF)/ 255.0) \
                                          alpha:a]
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- NavigationBar 高度
#define LEENavigationBarHeight ([[UIApplication sharedApplication] statusBarFrame].size.height + 44.0)
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- App 信息请求地址  以微信为例
#define LEERequestAppInfoUrlString @"http://itunes.apple.com/lookup?id=414478124"

#pragma mark -- App Store 跳转地址  以微信为例
#define LEEGotoAppStoreUrlString @"https://itunes.apple.com/cn/app/wei-xin/id414478124?mt=8"
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 在主线程中执行
#define LEEExcuteOnMain(block) !block ? : [[NSThread currentThread] isMainThread] ? block() : dispatch_async(dispatch_get_main_queue(), block)
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 通知
#define LEENotiObserver(observer, SEL, key, anObject) [[NSNotificationCenter defaultCenter] 
                 addObserver:observer \                                                                            
                    selector:SEL \                                                                                   
                        name:key \                                                                                   
                      object:anObject]
#define LEEPostNoti(key, anObject, aUserInfo) [[NSNotificationCenter defaultCenter] 
        postNotificationName:key \                                                                              
                       bject:anObject \
                    userInfo:aUserInfo]
#define LEERemoveNoti(observer, key, anObject) [[NSNotificationCenter defaultCenter] 
              removeObserver:observer \ 
                        name:key \                                                                                      
                      object:anObject]
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 字符串比较
#define isStringEqual(s1, s2) [s1 isEqualToString:s2]
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 当前软件语言
#define LEELanguage LEEShareConfig.language
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 根据当前语言选文本
#define LEELocation(key) NSLocalizedStringFromTable(key, LEELanguage, nil)
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 屏幕比例
#define kIPHONE_4 CGSizeEqualToSize([UIScreen mainScreen].bounds.size, CGSizeMake(320, 480))
#define kIPHONE_5_SE CGSizeEqualToSize([UIScreen mainScreen].bounds.size, CGSizeMake(320, 568))
#define kIPHONE_6_7_8 CGSizeEqualToSize([UIScreen mainScreen].bounds.size, CGSizeMake(375, 667))
#define kIPHONE_P CGSizeEqualToSize([UIScreen mainScreen].bounds.size, CGSizeMake(414, 736))
#define kIPHONE_X CGSizeEqualToSize([UIScreen mainScreen].bounds.size, CGSizeMake(375, 812))
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 弱变量转换
#define WS(sself) __weak __typeof(&*self)sself = self
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 占位图
#define LEEPlaceholderImg [UIImage imageNamed:@"Placeholder"]
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 获取图片资源
#define GetImage(imageName) [UIImage imageNamed:[NSString stringWithFormat:@"%@",imageName]]
{% endhighlight %}

---

{% highlight raw %}
#pragma mark -- 角度转弧度
#define DegreesToRadian(x) (M_PI * (x) / 180.0)

#pragma mark -- 弧度转换角度
#define RadianToDegrees(radian) (radian*180.0)/(M_PI)
{% endhighlight %}