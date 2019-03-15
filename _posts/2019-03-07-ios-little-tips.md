---
title: "iOS开发小知识总结"
layout: post
date: 2019-3-3 9:00
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

iOS开发中一些常见的小知识

---

#### 1、CocoaPods找不到头文件
- 问　　题：使用CocoaPods时，import找不到头文件。
- 原　　因：这是因为还没设置头文件的目录。
- 解决办法：TARGETS——Build Setting——User Header Search Paths，
           添加CocoaPods头文件目录，目录路径直接写：${SRCROOT}，
           后边选择recursive(会在相应的目录递归搜索文件) 。
- 注　　意：是在User Header Search Paths里添加，不是 Header Search Paths。

#### 2.CocoaPods导入头文件不提示
- 问　　题：使用CocoaPods时，import导入头文件时，不提示文件名。
- 原　　因：这是因为还没设置头文件的目录。
- 解决办法：TARGETS——Build Setting——User Header Search Paths，
        添加CocoaPods头文件目录，目录路径直接写：$(PODS_ROOT)，
        后边选择recursive (会在相应的目录递归搜索文件),就可以了。
- 注　　意：是在User Header Search Paths里添加，不是上面的Header Search Paths。

#### 3.跳进APP权限设置
{% highlight raw %}
NSURL *url = [NSURL URLWithString:UIApplicationOpenSettingsURLString];
if([[UIApplication sharedApplication] canOpenURL:url]) {
    if ([[[UIDevice currentDevice] systemVersion] floatValue] < 10.0) {
        // iOS 10.0 以下打开设置
        [[UIApplication sharedApplication] openURL:url];
    }else {
        // iOS 10.0 以上打开设置
        [[UIApplication sharedApplication] openURL:url
                                           options:[NSDictionary dictionary]
                                 completionHandler:^(BOOL success) {
        }];
    }
}
{% endhighlight %}


#### 4.强／弱引用
{% highlight raw %}
#define WeakSelf(type)  __weak typeof(type) weak##type = type; // weak
#define StrongSelf(type)  __strong typeof(type) type = weak##type; // strong
{% endhighlight %}

#### 5.获取图片资源
{% highlight raw %}
#define GetImage(imageName) [UIImage imageNamed:[NSString stringWithFormat:@"%@",imageName]]
{% endhighlight %}

#### 6.删除某个view所有的子视图
{% highlight raw %}
[[someView subviews] makeObjectsPerformSelector:@selector(removeFromSuperview)];
{% endhighlight %}

#### 7.设置tableView分割线颜色
{% highlight raw %}
[self.tableView setSeparatorColor:[UIColor grayColor]];
{% endhighlight %}

#### 8.消除tableViewCell分割线与屏幕边缘的距离
{% highlight raw %}
self.tableView.separatorInset = UIEdgeInsetsMake(0, 0, 0, 0);
{% endhighlight %}

#### 9.修改导航栏上返回按钮上的文字
{% highlight raw %}
UIBarButtonItem *backBtn = [[UIBarButtonItem alloc] init];
backBtn.title = @"返回";
//注意这里的self是父ViewController,不是即将显示的子ViewController
self.navigationItem.backBarButtonItem = backBtn;
{% endhighlight %}

#### 10.网络请求时状态栏转圈圈，开始请求时设置为YES，请求结束时置为NO
{% highlight raw %}
[UIApplication sharedApplication].networkActivityIndicatorVisible = YES;
{% endhighlight %}

#### 11.为项目添加全局PrefixHeader.pch预编译文件
在项目的配置界面选择TARGETS下的标签->Build Settings->搜索"prefix header"->配置“prefix header”的值为：
$(SRCROOT)/ProjectName/PrefixHeader.pch。其中$(SRCROOT)是项目根路径，ProjectName与实际的项目名一致，
PrefixHeader.pch与文件名一致。另：Precompile Prefix Header 设置为 YES。

#### 12.动画切换window的根控制器
{% highlight raw %}
// options是动画选项
[UIView transitionWithView:[UIApplication sharedApplication].keyWindow duration:0.5f options:UIViewAnimationOptionTransitionCrossDissolve animations:^{
        BOOL oldState = [UIView areAnimationsEnabled];
        [UIView setAnimationsEnabled:NO];
        [UIApplication sharedApplication].keyWindow.rootViewController = [RootViewController new];
        [UIView setAnimationsEnabled:oldState];
    } completion:^(BOOL finished) {
        // do something
}];
{% endhighlight %}

#### 13.去除数组中重复的对象
{% highlight raw %}
NSArray *newArr = [oldArr valueForKeyPath:@“@distinctUnionOfObjects.self"];
{% endhighlight %}

#### 14.注意定期清理SDWebImage的本地缓存，不然服务器更新原URL的图片时，APP仍然读取本地缓存中的图片而不是重新获取新图片。

#### 15.获取沙盒 Document
{% highlight raw %}
#define PathDocument [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject]
{% endhighlight %}

#### 16.获取沙盒 Cache
{% highlight raw %}
#define PathCache [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) firstObject]
{% endhighlight %}

#### 17.GCD代码只执行一次
{% highlight raw %}
#define kDISPATCH_ONCE_BLOCK(onceBlock) static dispatch_once_t onceToken; dispatch_once(&onceToken, onceBlock);
{% endhighlight %}

#### 18.自定义NSLog
{% highlight raw %}
#ifdef DEBUG
#define NSLog(fmt, ...) NSLog((@"%s [Line %d] " fmt), __PRETTY_FUNCTION__, __LINE__, ##__VA_ARGS__)
#else
#define NSLog(...)
#endif
{% endhighlight %}

#### 19.自定义Font
{% highlight raw %}
#define FontL(s)             [UIFont systemFontOfSize:s weight:UIFontWeightLight]
#define FontR(s)             [UIFont systemFontOfSize:s weight:UIFontWeightRegular]
#define FontB(s)             [UIFont systemFontOfSize:s weight:UIFontWeightBold]
#define FontT(s)             [UIFont systemFontOfSize:s weight:UIFontWeightThin]
#define Font(s)              FontL(s)
{% endhighlight %}

#### 20.FORMAT
{% highlight raw %}
#define FORMAT(f, ...)      [NSString stringWithFormat:f, ## __VA_ARGS__]
{% endhighlight %}

#### 21、在主线程上运行
{% highlight raw %}
#define kDISPATCH_MAIN_THREAD(mainQueueBlock) dispatch_async(dispatch_get_main_queue(), mainQueueBlock);
{% endhighlight %}

#### 22、开启异步线程
{% highlight raw %}
#define kDISPATCH_GLOBAL_QUEUE_DEFAULT(globalQueueBlock) dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), globalQueueBlocl);
{% endhighlight %}

#### 23.通知 
{% highlight raw %}
#define NOTIF_ADD(n, f)     [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(f) name:n object:nil]
#define NOTIF_POST(n, o)    [[NSNotificationCenter defaultCenter] postNotificationName:n object:o]
#define NOTIF_REMV()        [[NSNotificationCenter defaultCenter] removeObserver:self]
{% endhighlight %}

#### 24.获取window
{% highlight raw %}
+(UIWindow*)getWindow {
    UIWindow* win = nil;  //[UIApplication sharedApplication].keyWindow;
    for (id item in [UIApplication sharedApplication].windows) {
        if ([item class] == [UIWindow class]) {
            if (!((UIWindow*)item).hidden) {
                win = item;
                break;
            }
        }
    }
    return win;
}
{% endhighlight %}

#### 25.修改textField的placeholder的字体颜色、大小
{% highlight raw %}
[textField setValue:[UIColor redColor] forKeyPath:@"_placeholderLabel.textColor"];
[textField setValue:[UIFont boldSystemFontOfSize:16] forKeyPath:@"_placeholderLabel.font"];
{% endhighlight %}

#### 26.统一收起键盘
{% highlight raw %}
[[[UIApplication sharedApplication] keyWindow] endEditing:YES];
{% endhighlight %}

#### 27.控制屏幕旋转，在控制器中写
{% highlight raw %}
/** 是否支持自动转屏 */
- (BOOL)shouldAutorotate {
    return YES;
}
 
/** 支持哪些屏幕方向 */
- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskLandscapeLeft | UIInterfaceOrientationMaskLandscapeRight;
}
 
/** 默认的屏幕方向（当前ViewController必须是通过模态出来的UIViewController（模态带导航的无效）方式展现出来的，才会调用这个方法） */
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation {
    return UIInterfaceOrientationLandscapeLeft | UIInterfaceOrientationLandscapeRight;
}
{% endhighlight %}

#### 28.获取app缓存大小
{% highlight raw %}
- (CGFloat)getCachSize {    
    NSUInteger imageCacheSize = [[SDImageCache sharedImageCache] getSize];
    //获取自定义缓存大小
    //用枚举器遍历 一个文件夹的内容
    //1.获取 文件夹枚举器
    NSString *myCachePath = [NSHomeDirectory() stringByAppendingPathComponent:@"Library/Caches"];
    NSDirectoryEnumerator *enumerator = [[NSFileManager defaultManager] enumeratorAtPath:myCachePath];
    __block NSUInteger count = 0;
    //2.遍历
    for (NSString *fileName in enumerator) {
        NSString *path = [myCachePath stringByAppendingPathComponent:fileName];
        NSDictionary *fileDict = [[NSFileManager defaultManager] attributesOfItemAtPath:path error:nil];
        count += fileDict.fileSize;//自定义所有缓存大小
    }
    // 得到是字节  转化为M
    CGFloat totalSize = ((CGFloat)imageCacheSize+count)/1024/1024;
    return totalSize;
}
{% endhighlight %}

#### 29.清理app缓存
{% highlight raw %}
- (void)handleClearCache {
    //删除两部分
    //1.删除 sd 图片缓存
    //先清除内存中的图片缓存
    [[SDImageCache sharedImageCache] clearMemory];
    //清除磁盘的缓存
    [[SDImageCache sharedImageCache] clearDisk];
    //2.删除自己缓存
    NSString *myCachePath = [NSHomeDirectory() stringByAppendingPathComponent:@"Library/Caches"];
    [[NSFileManager defaultManager] removeItemAtPath:myCachePath error:nil];
}
{% endhighlight %}

#### 30.长按复制功能
{% highlight raw %}
- (void)viewDidLoad
{
    [self.view addGestureRecognizer:[[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(pasteBoard:)]];
}
- (void)pasteBoard:(UILongPressGestureRecognizer *)longPress {
    if (longPress.state == UIGestureRecognizerStateBegan) {
        UIPasteboard *pasteboard = [UIPasteboard generalPasteboard];
        pasteboard.string = @"需要复制的文本";
    }
}
{% endhighlight %}

#### 31.CocoaPods升级
{% highlight raw %}
在终端执行 sudo gem install -n / usr / local / bin cocoapods --pre
{% endhighlight %}

#### 32.判断图片类型
{% highlight raw %}
//通过图片Data数据第一个字节 来获取图片扩展名
- (NSString *)contentTypeForImageData:(NSData *)data
{
    uint8_t c;
    [data getBytes:&c length:1];
    switch (c)
    {
        case 0xFF:
            return @"jpeg";
 
        case 0x89:
            return @"png";
 
        case 0x47:
            return @"gif";
 
        case 0x49:
        case 0x4D:
            return @"tiff";
 
        case 0x52:
        if ([data length] < 12) {
            return nil;
        }
 
        NSString *testString = [[NSString alloc] initWithData:[data subdataWithRange:NSMakeRange(0, 12)] encoding:NSASCIIStringEncoding];
        if ([testString hasPrefix:@"RIFF"]
            && [testString hasSuffix:@"WEBP"])
        {
            return @"webp";
        }
 
        return nil;
    }
 
    return nil;
}
{% endhighlight %}

#### 33.获取手机和app信息
{% highlight raw %}
NSDictionary *infoDictionary = [[NSBundle mainBundle] infoDictionary];  
CFShow(infoDictionary);  
// app名称  
NSString *app_Name = [infoDictionary objectForKey:@"CFBundleDisplayName"];  
// app版本  
NSString *app_Version = [infoDictionary objectForKey:@"CFBundleShortVersionString"];  
// app build版本  
NSString *app_build = [infoDictionary objectForKey:@"CFBundleVersion"];    
      
//手机序列号  
NSString* identifierNumber = [[UIDevice currentDevice] uniqueIdentifier];  
NSLog(@"手机序列号: %@",identifierNumber);  
//手机别名： 用户定义的名称  
NSString* userPhoneName = [[UIDevice currentDevice] name];  
NSLog(@"手机别名: %@", userPhoneName);  
//设备名称  
NSString* deviceName = [[UIDevice currentDevice] systemName];  
NSLog(@"设备名称: %@",deviceName );  
//手机系统版本  
NSString* phoneVersion = [[UIDevice currentDevice] systemVersion];  
NSLog(@"手机系统版本: %@", phoneVersion);  
//手机型号  
NSString* phoneModel = [[UIDevice currentDevice] model];  
NSLog(@"手机型号: %@",phoneModel );  
//地方型号  （国际化区域名称）  
NSString* localPhoneModel = [[UIDevice currentDevice] localizedModel];  
NSLog(@"国际化区域名称: %@",localPhoneModel );  
      
NSDictionary *infoDictionary = [[NSBundle mainBundle] infoDictionary];  
// 当前应用名称  
NSString *appCurName = [infoDictionary objectForKey:@"CFBundleDisplayName"];  
NSLog(@"当前应用名称：%@",appCurName);  
// 当前应用软件版本  比如：1.0.1  
NSString *appCurVersion = [infoDictionary objectForKey:@"CFBundleShortVersionString"];  
NSLog(@"当前应用软件版本:%@",appCurVersion);  
// 当前应用版本号码   int类型  
NSString *appCurVersionNum = [infoDictionary objectForKey:@"CFBundleVersion"];  
NSLog(@"当前应用版本号码：%@",appCurVersionNum); 
{% endhighlight %}
    
### 34.获取一个类的所有属性
{% highlight raw %}
id LenderClass = objc_getClass("Lender");
unsigned int outCount, i;
objc_property_t *properties = class_copyPropertyList(LenderClass, &outCount);
for (i = 0; i < outCount; i++) {
    objc_property_t property = properties[i];
    fprintf(stdout, "%s %s\n", property_getName(property), property_getAttributes(property));
}
{% endhighlight %}

#### 35.image圆角
{% highlight raw %}
- (UIImage *)circleImage
{
    // NO代表透明
    UIGraphicsBeginImageContextWithOptions(self.size, NO, 1);
    // 获得上下文
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    // 添加一个圆
    CGRect rect = CGRectMake(0, 0, self.size.width, self.size.height);
    // 方形变圆形
    CGContextAddEllipseInRect(ctx, rect);
    // 裁剪
    CGContextClip(ctx);
    // 将图片画上去
    [self drawInRect:rect];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}
{% endhighlight %}

#### 36.view圆角
{% highlight raw %}
UIView *aView = [[UIView alloc] init]; 
aView.frame = CGRectMake(0, 0, 300, 200);
aView.backgroundColor = [UIColor redColor]; 
//设置圆角边框 
aView.layer.cornerRadius = 8; 
aView.layer.masksToBounds = YES;  
//设置边框及边框颜色  
aView.layer.borderWidth = 8; 
aView.layer.borderColor = [[UIColor grayColor] CGColor];
[self.view addSubview:aView];
{% endhighlight %}

#### 37.KVO监听某个对象的属性
{% highlight raw %}
// 添加监听者
[self addObserver:self forKeyPath:property options:NSKeyValueObservingOptionNew context:nil];
 
// 当监听的属性值变化的时候会来到这个方法
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    if ([keyPath isEqualToString:@"property"]) {
       [self textViewTextChange];
       } else {
     }
}
{% endhighlight %}

#### 38.AFNetworking监听网络状态
{% highlight raw %}
// 监听网络状况
AFNetworkReachabilityManager *mgr = [AFNetworkReachabilityManager sharedManager];
[mgr setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
    switch (status) {
        case AFNetworkReachabilityStatusUnknown:
            break;
        case AFNetworkReachabilityStatusNotReachable: {
            [SVProgressHUD showInfoWithStatus:@"当前设备无网络"];
        }
            break;
        case AFNetworkReachabilityStatusReachableViaWiFi:
            [SVProgressHUD showInfoWithStatus:@"当前Wi-Fi网络"];
            break;
        case AFNetworkReachabilityStatusReachableViaWWAN:
            [SVProgressHUD showInfoWithStatus:@"当前蜂窝移动网络"];
            break;
        default:
            break;
    }
}];
[mgr startMonitoring];
{% endhighlight %}

#### 39.判断字符串中是否有空格
{% highlight raw %}
+ (BOOL)isBlank:(NSString *)str {
    NSRange _range = [str rangeOfString:@" "];
    if (_range.location != NSNotFound) {
        //有空格
        return YES;
    } else {
        //没有空格
        return NO;
    }
}
{% endhighlight %}

#### 40.移除字符串中的空格和换行
{% highlight raw %}
+ (NSString *)removeSpaceAndNewline:(NSString *)str {
    NSString *temp = [str stringByReplacingOccurrencesOfString:@" " withString:@""];
    temp = [temp stringByReplacingOccurrencesOfString:@"\r" withString:@""];
    temp = [temp stringByReplacingOccurrencesOfString:@"\n" withString:@""];
    return temp;
}
{% endhighlight %}

#### 41.获取一个视频的第一帧图片
{% highlight raw %}
NSURL *url = [NSURL URLWithString:filepath];
AVURLAsset *asset1 = [[AVURLAsset alloc] initWithURL:url options:nil];
AVAssetImageGenerator *generate1 = [[AVAssetImageGenerator alloc] initWithAsset:asset1];
generate1.appliesPreferredTrackTransform = YES;
NSError *err = NULL;
CMTime time = CMTimeMake(1, 2);
CGImageRef oneRef = [generate1 copyCGImageAtTime:time actualTime:NULL error:&err];
UIImage *firstPic = [[UIImage alloc] initWithCGImage:oneRef];

return firstPic;
{% endhighlight %}

#### 42.获取视频的时长
{% highlight raw %}
+ (NSInteger)getVideoTimeByUrlString:(NSString *)urlString {
    NSURL *videoUrl = [NSURL URLWithString:urlString];
    AVURLAsset *avUrl = [AVURLAsset assetWithURL:videoUrl];
    CMTime time = [avUrl duration];
    int seconds = ceil(time.value/time.timescale);
    return seconds;
}
{% endhighlight %}

#### 43.字符串是否为空
{% highlight raw %}
+ (BOOL)isEqualToNil:(NSString *)str {
    return str.length <= 0 || [str isEqualToString:@""] || !str;
}
{% endhighlight %}

#### 44.当tableView占不满一屏时，去除下边多余的单元格
{% highlight raw %}
self.tableView.tableHeaderView = [UIView new];
self.tableView.tableFooterView = [UIView new];
{% endhighlight %}

#### 45.isKindOfClass 和 isMemberOfClass的区别
{% highlight raw %}
isKindOfClass可以判断某个对象是否属于某个类，或者这个类的子类。
isMemberOfClass更加精准，它只能判断这个对象类型是否为这个类(不能判断子类)
{% endhighlight %}

#### 46.从NSURL中拿到链接字符串
{% highlight raw %}
NSString *urlString = myURL.absoluteString;
{% endhighlight %}

#### 47.自定义cell选中背景颜色
{% highlight raw %}
UIView *bgColorView = [[UIView alloc] init];
bgColorView.backgroundColor = [UIColor redColor];
[cell setSelectedBackgroundView:bgColorView];
{% endhighlight %}

#### 48.获取图片大小
{% highlight raw %}
CGFloat imageWidth = image.size.width;
CGFloat imageHeight = imageWidth * image.scale;
{% endhighlight %}

#### 49.获取view的坐标在整个window上的位置
{% highlight raw %}
// v上的(0, 0)点在toView上的位置
CGPoint point = [v convertPoint:CGPointMake(0, 0) toView:[UIApplication sharedApplication].windows.lastObject];
或者
CGPoint point = [v.superview convertPoint:v.frame.origin toView:[UIApplication sharedApplication].windows.lastObject];
{% endhighlight %}

#### 50.在非ViewController的地方弹出UIAlertController对话框
{% highlight raw %}
//  最好抽成一个分类
UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"Title" message:@"message" preferredStyle:UIAlertControllerStyleAlert];
id rootViewController = [UIApplication sharedApplication].delegate.window.rootViewController;
if ([rootViewController isKindOfClass:[UINavigationController class]]) {
    rootViewController = ((UINavigationController *)rootViewController).viewControllers.firstObject;
}
if ([rootViewController isKindOfClass:[UITabBarController class]]) {
    rootViewController = ((UITabBarController *)rootViewController).selectedViewController;
}
[rootViewController presentViewController:alertController animated:YES completion:nil];
{% endhighlight %}

#### 51.在应用中打开设置的某个界面
{% highlight raw %}
// 打开设置->通用
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"prefs:root=General"]];

// 以下是设置其他界面
prefs:root=General&path=About
prefs:root=General&path=ACCESSIBILITY
prefs:root=AIRPLANE_MODE
prefs:root=General&path=AUTOLOCK
prefs:root=General&path=USAGE/CELLULAR_USAGE
prefs:root=Brightness
prefs:root=Bluetooth
prefs:root=General&path=DATE_AND_TIME
prefs:root=FACETIME
prefs:root=General
prefs:root=General&path=Keyboard
prefs:root=CASTLE
prefs:root=CASTLE&path=STORAGE_AND_BACKUP
prefs:root=General&path=INTERNATIONAL
prefs:root=LOCATION_SERVICES
prefs:root=ACCOUNT_SETTINGS
prefs:root=MUSIC
prefs:root=MUSIC&path=EQ
prefs:root=MUSIC&path=VolumeLimit
prefs:root=General&path=Network
prefs:root=NIKE_PLUS_IPOD
prefs:root=NOTES
prefs:root=NOTIFICATIONS_ID
prefs:root=Phone
prefs:root=Photos
prefs:root=General&path=ManagedConfigurationList
prefs:root=General&path=Reset
prefs:root=Sounds&path=Ringtone
prefs:root=Safari
prefs:root=General&path=Assistant
prefs:root=Sounds
prefs:root=General&path=SOFTWARE_UPDATE_LINK
prefs:root=STORE
prefs:root=TWITTER
prefs:root=FACEBOOK
prefs:root=General&path=USAGE prefs:root=VIDEO
prefs:root=General&path=Network/VPN
prefs:root=Wallpaper
prefs:root=WIFI
prefs:root=INTERNET_TETHERING
prefs:root=Phone&path=Blocked
prefs:root=DO_NOT_DISTURB
{% endhighlight %}


#### 52.仿苹果抖动动画
{% highlight raw %}
#define RADIANS(degrees) (((degrees) * M_PI) / 180.0)

- (void)startAnimate {
    view.transform = CGAffineTransformRotate(CGAffineTransformIdentity, RADIANS(-5));
    
    [UIView animateWithDuration:0.25 delay:0.0 options:(UIViewAnimationOptionAllowUserInteraction | UIViewAnimationOptionRepeat | UIViewAnimationOptionAutoreverse) animations:^ {
                         view.transform = CGAffineTransformRotate(CGAffineTransformIdentity, RADIANS(5));
                     } completion:nil];
}

- (void)stopAnimate {
    [UIView animateWithDuration:0.25 delay:0.0 options:(UIViewAnimationOptionAllowUserInteraction | UIViewAnimationOptionBeginFromCurrentState | UIViewAnimationOptionCurveLinear) animations:^ {
                         view.transform = CGAffineTransformIdentity;
                     } completion:nil];
}
{% endhighlight %}

#### 53.单个页面多个网络请求的情况，需要监听所有网络请求结束后刷新UI
{% highlight raw %}
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t serialQueue = dispatch_queue_create("com.wzb.test.www", DISPATCH_QUEUE_SERIAL);
dispatch_group_enter(group);
dispatch_group_async(group, serialQueue, ^{
    // 网络请求一
    [WebClick getDataSuccess:^(ResponseModel *model) {
        dispatch_group_leave(group);
    } failure:^(NSString *err) {
        dispatch_group_leave(group);
    }];
});
dispatch_group_enter(group);
dispatch_group_async(group, serialQueue, ^{
    // 网络请求二
    [WebClick getDataSuccess:getBigTypeRM onSuccess:^(ResponseModel *model) {
        dispatch_group_leave(group);
    } failure:^(NSString *errorString) {
        dispatch_group_leave(group);
    }];
});
dispatch_group_enter(group);
dispatch_group_async(group, serialQueue, ^{
    // 网络请求三
    [WebClick getDataSuccess:^{
        dispatch_group_leave(group);
    } failure:^(NSString *errorString) {
        dispatch_group_leave(group);
    }];
});
    
// 所有网络请求结束后会来到这个方法
dispatch_group_notify(group, serialQueue, ^{
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        dispatch_async(dispatch_get_main_queue(), ^{
            // 刷新UI
        });
    });
});
{% endhighlight %}