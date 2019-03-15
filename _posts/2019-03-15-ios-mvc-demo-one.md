---
title: "iOS简单的MVC模式实现(一)"
layout: post
date: 2019-3-15 11:00
image: #/assets/images/markdown.jpg
headerImage: false
tag:
- iOS
projects: true
hidden: false
category: project
author: #Yuan Lee
description: #Markdown summary with different options
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="http://localhost:4000/assets/images/profile.jpg" height="20" width="20" align="absmiddle">'
---

## Summary

        MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)
    的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集
    到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。
    
        本文将简单介绍 iOS 下 MVC模式 的实现，以及SDWebimage，MJRefresh，SDAutoLayout的
    简单使用，此外还有加载大图时避免占用过多内存的处理方法。


MJRefresh的使用在第三部分，SDWebImage 以及 SDAutoLayout 的使用将在[下一篇文章][1]介绍。

Demo Github 地址: [https://github.com/yuanlee0214/iOS-MVC-Demo][2]

#### 目录
- [Ⅰ.Model 部分](#model)
- [Ⅱ.View 部分](#view)
- [Ⅲ.Controller 部分](#controller)

<h3 id="model">一、Model 部分</h3>

设置 TableViewCell 的 Model

<h5>头文件 "LEECellModel.h"</h5>
{% highlight raw %}
#import <Foundation/Foundation.h>

@interface LEECellModel : NSObject

@property (nonatomic, copy) NSString *title;
@property (nonatomic, copy) NSString *imgName;

- (instancetype)initWithTitle:(NSString *)title
                      imgName:(NSString *)imgName;

@end
{% endhighlight %}

<h5>实现文件 "LEECellModel.m"</h5>
{% highlight raw %}
#import "LEECellModel.h"

@implementation LEECellModel
- (instancetype)initWithTitle:(NSString *)title
                      imgName:(NSString *)imgName {
    if (self = [super init]) {
        _title = title;
        _imgName = imgName;
    }
    return self;
}

@end
{% endhighlight %}

<div class="breaker"></div>

<h3 id="view">二、View 部分</h3>

设置 TableViewCell 的 样式

<h5>头文件 "LEETableViewCell.h"</h5>
{% highlight raw %}
#import <UIKit/UIKit.h>
#import "LEECellModel.h"

@interface LEETableViewCell : UITableViewCell
@property (nonatomic, weak) UIImageView *imgView;
@property (nonatomic, weak) UILabel *titleLabel;

/**
 界面布局
 */
- (void)layoutView;

/**
 设置模型
 */
- (void)setModel:(LEECellModel *)model;

@end
{% endhighlight %}

<h5>实现文件 "LEETableViewCell.m"</h5>
{% highlight raw %}
#import "LEETableViewCell.h"

@implementation LEETableViewCell

- (instancetype)initWithStyle:(UITableViewCellStyle)style
              reuseIdentifier:(NSString *)reuseIdentifier {
    self = [super initWithStyle:style
                reuseIdentifier:reuseIdentifier];
    if (self) {
        [self layoutView];
    }
    return self;
}

- (void)layoutView {
    
    // titleLabel  cell的标题显示在左侧
    UILabel *titleLabel = [[UILabel alloc] initWithFrame:CGRectMake(autoScaleH(15), 0, autoScaleH(100), autoScaleV(57.5))];
    titleLabel.font = [UIFont systemFontOfSize:autoScaleF(19)];
    titleLabel.textColor = SHBHexColor(0x333333, 1.0);
    titleLabel.textAlignment = NSTextAlignmentLeft;
    titleLabel.centerY = autoScaleV(57.5) / 2.0;
    [self.contentView addSubview:titleLabel];
    _titleLabel = titleLabel;
    
    // imgV  自定义cell的图片显示在右侧
    UIImageView *imgView = [[UIImageView alloc] init];
    imgView.frame = CGRectMake(SHBSWidth - autoScaleH(100), 0, autoScaleH(81), autoScaleV(57.5));
    imgView.centerY = autoScaleV(57.5) / 2.0;
    [self.contentView addSubview:imgView];
    _imgView = imgView;
}

- (void)setModel:(LEECellModel *)model {
    UIImage *image;
    if (model.imgName.length > 0) {
        image = [UIImage imageNamed:model.imgName];
    }
    _imgView.image = image;

    _titleLabel.text = model.title;
}

@end
{% endhighlight %}

<div class="breaker"></div>

<h3 id="controller">三、Controller 部分</h3>

设置 MainViewController

<h5>头文件 "MainViewController.h"</h5>
{% highlight raw %}
#import <UIKit/UIKit.h>

@interface MainViewController : UIViewController

@end
{% endhighlight %}

<h5>实现文件 "MainViewController.m"</h5>
{% highlight raw %}
#import "MJRefresh.h"
#import "LEEDetailVC.h"
#import "LEECellModel.h"
#import "LEETableViewCell.h"
#import "MainViewController.h"

@interface MainViewController () <UITableViewDelegate, UITableViewDataSource>
@property (nonatomic, weak) UITableView *tableView;
@end

@implementation MainViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self layoutView];
}

#pragma mark -- TableView Delegate
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    return 3;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    NSInteger num = 0;
    if (section == 0) {
        num = 3;
    } else if (section == 1) {
        num = 2;
    } else { // 2
        num = 1;
    }
    return num;
}

- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section {
    return 35;
}

- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    return 50;
}

- (CGFloat)tableView:(UITableView *)tableView heightForFooterInSection:(NSInteger)section {
    return CGFLOAT_MIN;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    LEETableViewCell *cell;
    cell = [tableView dequeueReusableCellWithIdentifier:NSStringFromClass([LEETableViewCell class])];
    if (!cell) {
        cell = [[LEETableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:NSStringFromClass([LEETableViewCell class])];
    }
    LEECellModel *model;
    if (indexPath.section == 0) {
        if (indexPath.row == 0) {
            model = [[LEECellModel alloc] initWithTitle:@"NetImg 1"
                                                imgName:@"pic1"];
        } else if (indexPath.row == 1) {
            model = [[LEECellModel alloc] initWithTitle:@"NetImg 2"
                                                imgName:@"pic2"];
        } else if (indexPath.row == 2) {
            model = [[LEECellModel alloc] initWithTitle:@"NetImg 3"
                                                imgName:@"pic3"];
        }
    } else if (indexPath.section == 1) {
        if (indexPath.row == 0) {
            model = [[LEECellModel alloc] initWithTitle:@"LocalImg 1"
                                                imgName:@"pic4"];
        } else { // 1
            model = [[LEECellModel alloc] initWithTitle:@"LocalImg 2"
                                                imgName:@"pic5"];
        }
    } else { // 2
        model = [[LEECellModel alloc] initWithTitle:@"NetLongImg"
                                            imgName:@"pic6"];
    }
    [cell setModel:model];
    return cell;
}

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
    LEEDetailVC *detailVC = [[LEEDetailVC alloc] init];
    if (indexPath.section == 0) {  // 加载网络大图的三个cell
        if (indexPath.row == 0) {
            detailVC.imgKey = @"NetImg 1";
            detailVC.imgURL = @"https://get.pxhere.com/photo/horizon-structure-sky-highway-stadium-atmosphere-of-earth-sport-venue-57501.jpg";
        } else if (indexPath.row == 1) {
            detailVC.imgKey = @"NetImg 2";
            detailVC.imgURL = @"http://www.wanwenku.com/uploadfile/2016/0528/20160528125916910.jpg";
        } else if (indexPath.row == 2) {
            detailVC.imgKey = @"NetImg 3";
            detailVC.imgURL = @"http://res.downhot.com/d/file/p/2014/07/20/418aba73e413176ba3ec9f8f2df5a465.jpg";
        }
    } else if (indexPath.section == 1) {  // 加载本地大图的两个cell
        if (indexPath.row == 0) {
            detailVC.imgKey = @"LocalImg 1";
            detailVC.imgName = @"BigPic1";  // 这张图片10M, 加载起来很慢
        } else {
            detailVC.imgKey = @"LocalImg 2";
            detailVC.imgName = @"BigPic2";  // 这张图片5M, 加载起来速度尚可
        }
    } else { // 2  加载网络长图的一个cell
        detailVC.imgKey = @"LongImg";
        detailVC.imgURL = @"http://ww2.sinaimg.cn/large/bd2fd49bgw1e26kv4tyqwj.jpg";
    }
    [self.navigationController pushViewController:detailVC animated:YES];
}

- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section {
    
    UIView *headerView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, LEESWidth, 35)];
    headerView.backgroundColor = LEEHexColor(0xeeeeee, 1.0);
    [self.view addSubview:headerView];
    
    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(20, 0, LEESWidth - 30, 35)];
    label.text = @"";
    label.textColor = LEEHexColor(0x333333, 1.0);
    label.font = [UIFont systemFontOfSize:17];
    label.textAlignment = NSTextAlignmentLeft;
    [headerView addSubview:label];
    
    if (section == 0) {
        UIView *line = [[UIView alloc] initWithFrame:CGRectMake(0, 0, LEESWidth, 0.5)];
        line.backgroundColor = LEEHexColor(0xcccccc, 1.0);
        [headerView addSubview:line];
        label.text = @"网络大图";
    } else if (section == 1) {
        label.text = @"本地大图";
    } else { // 2
        label.text = @"网络长图";
    }
    
    return headerView;
}

- (UIView *)tableView:(UITableView *)tableView viewForFooterInSection:(NSInteger)section {
    return [[UIView alloc] init];
}

#pragma mark -- 公有方法
- (void)layoutView {
    self.view.backgroundColor = LEEHexColor(0xffffff, 1.0);
    self.title = @"YuanLee";
    
    UIBarButtonItem *backBtn = [[UIBarButtonItem alloc] init];
    backBtn.title = @"返回";
    // 修改导航栏上返回按钮上的文字，注意这里的self是父ViewController,不是即将显示的子ViewController
    self.navigationItem.backBarButtonItem = backBtn;
    
    UITableView *tableView = [[UITableView alloc] initWithFrame:self.view.bounds
                                                          style:UITableViewStyleGrouped];
    tableView.delegate = self;
    tableView.dataSource = self;
    // 消除 cell 的线与屏幕边缘的距离
    tableView.separatorInset = UIEdgeInsetsMake(0, 0, 0, 0);
    tableView.backgroundColor = LEEHexColor(0xffffff, 1.0);
    tableView.separatorColor = LEEHexColor(0xcccccc, 1.0);
    [self.view addSubview:tableView];
    _tableView = tableView;
    
    // 使用 MJRefresh 实现 TableView 的下拉刷新
    // 设置回调（一旦进入刷新状态，就调用target的action，也就是调用self的loadNewData方法）
    MJRefreshNormalHeader *header = [MJRefreshNormalHeader headerWithRefreshingTarget:self refreshingAction:@selector(refreshData)];
    //提示内容
    [header setTitle:@"Pull Down To Refresh" forState:MJRefreshStateIdle];
    [header setTitle:@"Release To Refresh" forState:MJRefreshStatePulling];
    [header setTitle:@"Refreshing" forState:MJRefreshStateRefreshing];
    // 设置自动切换透明度(在导航栏下面自动隐藏)
    header.automaticallyChangeAlpha = YES;
    // 隐藏时间
    header.lastUpdatedTimeLabel.hidden = YES;
    // 马上进入刷新状态
    [header beginRefreshing];
    // 设置header
    self.tableView.mj_header = header;
}

- (void)viewUpdate {
    [_tableView reloadData];
    [(MJRefreshNormalHeader *)_tableView.mj_header setTitle:@"Pull Down To Refresh" forState:MJRefreshStateIdle];
    [(MJRefreshNormalHeader *)_tableView.mj_header setTitle:@"Release To Refresh" forState:MJRefreshStatePulling];
    [(MJRefreshNormalHeader *)_tableView.mj_header setTitle:@"Refreshing" forState:MJRefreshStateRefreshing];
}

- (void)refreshData
{
    //2秒后刷新表格UI
    __weak UITableView *tableView = self.tableView;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        // 刷新表格
        [tableView reloadData];
        
        // 拿到当前的下拉刷新控件，结束刷新状态
        [tableView.mj_header endRefreshing];
    });
}

@end
{% endhighlight %}

[1]: https://www.yuanlee.cc/ios-mvc-demo-two/
[2]: https://github.com/yuanlee0214/iOS-MVC-Demo

