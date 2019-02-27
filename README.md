# SFScaleImageView

[![CI Status](https://img.shields.io/travis/liuxuleidota/SFScaleImageView.svg?style=flat)](https://travis-ci.org/liuxuleidota/SFScaleImageView)
[![Version](https://img.shields.io/cocoapods/v/SFScaleImageView.svg?style=flat)](https://cocoapods.org/pods/SFScaleImageView)
[![License](https://img.shields.io/cocoapods/l/SFScaleImageView.svg?style=flat)](https://cocoapods.org/pods/SFScaleImageView)
[![Platform](https://img.shields.io/cocoapods/p/SFScaleImageView.svg?style=flat)](https://cocoapods.org/pods/SFScaleImageView)

## Example

To run the example project, clone the repo, and run `pod install` from the Example directory first.

## Requirements

## Installation

SFScaleImageView is available through [CocoaPods](https://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod 'SFScaleImageView'
```

## 总体效果：
![Dec-29-2018 16-41-08.gif](https://github.com/liuxuleidota/SFScaleImageView/blob/master/Dec-29-2018%2016-41-08.gif)

## 步骤如下：
## 一、初始化各view，添加事件
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    /*首先，本文要实现的效果类似于微信个人中心中查看自己的头像，具体功能点如下：
     1.进入界面原始状态为image居中显示
     2.当scale==1时，双击放大图片到：高度等于屏幕高度，同时等比缩放宽度，放大后，竖直方向不可滚动，水平可以
     3.当scale>1时，双击复原
     4.捏合手势缩放后，zoomScale在min与max之间，这时保持imageView竖直居中
     */
    UIScrollView *scrollView = [[UIScrollView alloc] init];
    _scrollV = scrollView;
    
    scrollView.backgroundColor = UIColor.blackColor;
    scrollView.delegate = self;
    [self.view addSubview:scrollView];
    
    UITapGestureRecognizer *tapGes = [[UITapGestureRecognizer alloc] initWithTarget:self
                                                                             action:@selector(doubleTapScrollV:)];
    tapGes.numberOfTapsRequired = 2;
    [scrollView addGestureRecognizer:tapGes];
    
    //这里有个小tips，[[UIImageView alloc] initWithImage:]这个方法，
    //初始化出来的imageView.frame是等于(0, 0, image.size.width, image.size.height)的
    //之前一直以为是CGRectZero...
    _imageView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"soccer"]];
    [scrollView addSubview:self.imageView];
    
    //想要做到放大后刚好竖直方向撑满屏幕，那么maxScale就等于SCREEN_HEIGHT/SCREEN_WIDTH
    CGFloat maxScale = SCREEN_HEIGHT/SCREEN_WIDTH;
    scrollView.maximumZoomScale = maxScale;
    scrollView.minimumZoomScale = 1;
    //scrollView是全屏大小
    scrollView.frame = UIScreen.mainScreen.bounds;
    
    //imageView居中,与scrollView等宽
    _imageView.frame = CGRectMake(0, 0, scrollView.frame.size.width, scrollView.frame.size.width);
    _imageView.center = _scrollV.center;
}
```
## 二、返回需要缩放的子视图
```
//返回需要缩放的子视图
- (UIView *)viewForZoomingInScrollView:(UIScrollView *)scrollView{
    return self.imageView;
}
```
## 三、scrollView进行了缩放
```
//scrollView进行了缩放
//通过方法zoomToRect来缩放时，这个方法只会调用一次，调用时间点在scrollViewWillBeginZooming之后，scrollViewDidEndZooming之前
//使用双指捏合手势缩放中，这个方法会一直调用,缩放中,scrollView的contentSize会变化,即放大缩小
//如果内容宽(高)小于scrollView宽(高),内容以scrollView宽(高)为准居中
//如果内容宽(高)大于scrollView宽(高),内容从x=0(y=0)开始
//如此便达到了,内容任一方向小于父容器时,居中该方向,
//大于父容器时,内容占满该方向
- (void)scrollViewDidZoom:(UIScrollView *)scrollView{
    UIView *targetView = [self viewForZoomingInScrollView:scrollView];
    BOOL widthIsSmall = targetView.width < scrollView.width;
    BOOL heightIsSmall = targetView.height < scrollView.height;
    
    if (widthIsSmall) {
        targetView.centerX = scrollView.centerX;
    } else {
        targetView.left = 0;
    }
    
    if (heightIsSmall) {
        targetView.centerY = scrollView.centerY;
    } else {
        targetView.top = 0;
    }
}
```
## 四、处理点击事件
```
- (void)doubleTapScrollV:(UITapGestureRecognizer*)tapGes{
    CGFloat currScale = self.scrollV.zoomScale;
    CGFloat minScale = self.scrollV.minimumZoomScale;
    CGFloat maxScale = self.scrollV.maximumZoomScale;
    CGFloat goalScale;
    
    if (currScale == minScale) {
        goalScale = maxScale;
    } else {
        goalScale = minScale;
    }
    
    if (currScale == goalScale) {
        return;
    }
    
    CGPoint touchPoint = [tapGes locationInView:[self viewForZoomingInScrollView:self.scrollV]];
    //缩放后的目标frame大小,当然是通过容器(scrollView)的size/scale来计算的,如下:
    CGFloat xsize = self.scrollV.frame.size.width/goalScale;
    CGFloat ysize = self.scrollV.frame.size.height/goalScale;
    //origin当然是点击点减去一半的宽高,
    //x,y可能小于0,但是没问题,因为内容不能从负值开始,如下
    CGFloat x = touchPoint.x-xsize/2;
    CGFloat y = touchPoint.y-ysize/2;
    
    [self.scrollV zoomToRect:CGRectMake(x, y, xsize, ysize) animated:YES];
}
```

## Author

liuxuleidota, liu_xu_lei@163.com

## License

SFScaleImageView is available under the MIT license. See the LICENSE file for more info.
