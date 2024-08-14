---
title: 好奇 离屏渲染
date: 2023-09-20 08:00:00 +0800
categories: [ios]
tags: [OC]
---

## 为啥要知道离屏渲染？

因为它要在当前屏幕外开辟另一个缓冲区进行图形渲染，并且涉及到帧缓冲区和离屏缓冲区的频繁切换，影响 GPU 性能，每隔 16.7ms 的帧信号到来之前没有完成画面的合成，最终造成卡顿、掉帧。

## 那什么情况会触发离屏渲染呢？

- 圆角
- 图层蒙版
- 阴影
- 光栅化

## 圆角这些为啥不能在屏渲染？

标准的屏幕渲染（On-Screen Rendering）是按照一定顺序将各个图层绘制到帧缓冲区（Frame Buffer）中，这个过程中，一旦图层被绘制，其数据就被用于显示，无法再次修改。而当需要实现圆角这样的效果时，可能需要对已经绘制的内容进行裁剪，这就需要在屏幕外的另一个缓冲区进行渲染，然后再将结果合并到屏幕上，从而产生了离屏渲染的需求。

## 那我改怎么设置圆角？

我们参考 YYImage 是怎么做的：

```objective-c
- (UIImage *)yy_imageByRoundCornerRadius:(CGFloat)radius
                                corners:(UIRectCorner)corners
                           borderWidth:(CGFloat)borderWidth
                          borderColor:(UIColor *)borderColor
                        borderLineJoin:(CGLineJoin)borderLineJoin {
    // 省略部分代码...
    UIGraphicsBeginImageContextWithOptions(self.size, NO, self.scale);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGRect rect = CGRectMake(0, 0, self.size.width, self.size.height);
    CGContextScaleCTM(context, 1, -1);
    CGContextTranslateCTM(context, 0, -rect.size.height);
    CGFloat minSize = MIN(self.size.width, self.size.height);
    if (borderWidth < minSize / 2) {
        UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:CGRectInset(rect, borderWidth, borderWidth)
                                                           byRoundingCorners:corners
                                                             cornerRadii:CGSizeMake(radius, borderWidth)];
        [path closePath];
        CGContextSaveGState(context);
        [path addClip];
        CGContextDrawImage(context, rect, self.CGImage);
        CGContextRestoreGState(context);
    }
    if (borderColor && borderWidth < minSize / 2 && borderWidth > 0) {
        // 省略部分代码...
    }
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}
```
