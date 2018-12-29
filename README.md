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
- (void)scrollViewDidZoom:(UIScrollView *)scrollView{
    if (scrollView.zoomScale > 1) {
        //在手势缩放过程中，
        //imageWidth一直大于screenWidth,但是imageWidth一直等于scrollView.contentSize.width，
        //所以一直保持_imageView的center.x等于contentSize.width/2
        //imageHeight一直<=scrollView.height，所以要保持_imageView的center.y等于scrollView.height/2
        _imageView.center = CGPointMake(scrollView.contentSize.width/2, scrollView.frame.size.height/2);
    } else {
        _imageView.center = scrollView.center;
    }
}
```
## 四、处理点击事件
```
/*
 以点击点为中心，计算需要缩放查看的区域
 这个方法要返回的坐标系是对于viewForZoomingInScrollView方法返回的view来说，这一点要理解！在本例中即为imageView
 */
- (CGRect)getRectWithScale:(CGFloat)scale andCenter:(CGPoint)center{
    CGFloat oldWidth = _imageView.frame.size.width;
    //newWidth等于oldWidth（scale为1时）除以目标scale，
    //解释：当目标scale>1时，newWidth小于原宽度，而新的宽度会被用来充满scroll.width，所以导致了图片局部放大
    CGFloat newWidth = oldWidth/scale;
    //newHeight等于imageView.height，因为要在竖直方向上撑满
    CGFloat newHeight = _imageView.frame.size.height;
    //newX等于center.x - newWidth/2，这里可能会出现新的x小于0或view.right大于superView.width
    //但是没关系，因为view.frame.origin.x必须在（0, superView.width-view.width）之间
    CGFloat newX = center.x - newWidth/2;
    //竖直方向撑满，意味着newY从0开始
    CGFloat newY = 0;
    return CGRectMake(newX, newY, newWidth, newHeight);
}

- (void)doubleTapScrollV:(UITapGestureRecognizer*)tapGes{
    if (_scrollV.zoomScale > 1) {
        [_scrollV setZoomScale:1 animated:YES];
    } else if (_scrollV.zoomScale == 1){
        CGPoint newCenter = [tapGes locationInView:_imageView];
        CGRect goalRect = [self getRectWithScale:_scrollV.maximumZoomScale andCenter:newCenter];
        [_scrollV zoomToRect:goalRect animated:YES];
    }
}
```

## Author

liuxuleidota, liu_xu_lei@163.com

## License

SFScaleImageView is available under the MIT license. See the LICENSE file for more info.
