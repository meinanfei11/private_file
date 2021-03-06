---
title: Android 屏幕适配
tag: Android
categories: 适配
---



## 为什么要适配

由于 Android 系统的开放性，任何用户、开发者、 OEM 厂商、运营商都可以对 Android 进行定制，于是导致运行 Android 的设备多种多样，他们有着不同的屏幕尺寸和像素密度。

尽管系统可以通过基本的缩放和调整大小使得界面适配不同的屏幕，但进一步优化，可以确保界面能够在各类屏幕上美观的呈现。

<!-- more -->

![image-20210128142240773](https://raw.githubusercontent.com/xiaomanwong/static_file/master/images/image-20210128142240773.png)

## 基本概念

### 屏幕尺寸

屏幕尺寸指 **屏幕对角线的长度**，单位是 *英寸*，一英寸 = 2.54 厘米

> 常见的 Android 设备尺寸有 2.4 、 2.8、3.5、3.7、4.2、5.0、5.5、6.0，目前 5.5 以上的设备偏多

### 屏幕分辨率

屏幕分辨率是指在 横纵方向上的像素点数，单位是 *px* ，1px = 1个像素点。

一般以 纵向像素 * 横向像素，如 1920 * 1080 来表示，即 宽度方向上有 1080 个像素点，高度方向上有 1920 个像素点。

> 单位:  px(pixel) ，1 px = 1像素点
>
> Android 手机常见的分辨率： 320 * 480    480 * 800  720 * 1280  1080 * 1920
>
> UI 设计图一般会以 px 作为统一的计量单位

### 屏幕像素密度

屏幕像素密度是指 **每英寸上的像素点**， 单位是 *dpi*, 即 “dot per inch” 的缩写。

屏幕像素密度与屏幕尺寸和屏幕分辨率有关，在单一变化条件下，屏幕尺寸越小，分辨率越高，像素密度越大，反之越小。dp = px / inch

> 假设设备内每英寸有 160 个像素点，那么该设备的屏幕像素密度 = 160 dpi

Android 手机对每类手机屏幕大小都有一个相应的屏幕像素密度

| 密度类型             | 代表的分辨率 px | 屏幕像素密度 dpi |
| -------------------- | --------------- | ---------------- |
| 低密度（ldpi）       | 240 * 320       | 120              |
| 中密度（mdpi）       | 320 * 480       | 160              |
| 高密度（hdpi）       | 480 * 800       | 240              |
| 超高密度（xhdpi）    | 720 * 1280      | 320              |
| 超超高密度（xxhdpi） | 1080 * 1920     | 480              |

### 屏幕尺寸、分辨率、像素密度三者关系

一部手机的分辨率是 **宽 x 高**， 屏幕大小是以寸为单位，三者关系为:

密度 dp = 像素 px / 屏幕大小 inch

密度（dpi） = $\frac {\sqrt{宽^2 + 高^2}}{屏幕大小} $

1. 密度即每英寸的像素点
2. 勾股定理求出手机的对角线物理尺寸
3. 再储以屏幕大小即可

![image-20210128150307492](https://raw.githubusercontent.com/xiaomanwong/static_file/master/images/image-20210128150307492.png)

### 密度无关像素

`density-independent pixel` 叫做 `dp`  或 `dip` ，与终端上的实际物理像素点无关。可以保证在不同屏幕像素密度的设备上显示相同的效果。

> Android 开发时用 dp 而不是 px 单位设置图片大小，是  Android 特有的单位
>
> 场景：假如同样是画一条屏幕一半的线，如果使用 px 作为单位，那么在 480 * 800 分辨率的设备上应为 240 px. 在 320 * 480 的设备上设置为 160 px。
>
> 如果使用 dp 为单位，在两种分辨率下，  160dp 都显示为屏幕一半的长度。

### dp 与 px 的转换

`px = dp * (dpi / 160)`

| 密度类型          | 代表的分辨率 px | 屏幕密度 dpi | 换算（px/dp) | 比例 |
| ----------------- | --------------- | ------------ | ------------ | ---- |
| 低密度 ldpi       | 240 x 320       | 120          | 1dp = 0.75px | 3    |
| 中密度 mdpi       | 320 x 480       | 160          | 1dp = 1px    | 4    |
| 高密度 hdpi       | 480 x 800       | 240          | 1dp = 1.5px  | 6    |
| 超高密度 xhdpi    | 720 x 1280      | 320          | 1dp = 2px    | 8    |
| 超超高密度 xxhdpi | 1080 x 1920     | 480          | 1dp = 3px    | 12   |

在 Android 中，规定 以 `160dpi` 即屏幕分辨率为 320 x 480 为基准：1 dp = 1 px

### 独立比例像素

`sp`, `scale-independent pixels`, 与 dp 类似，但是可以根据文字大小首选项进行缩放，是设置字体大小的御用单位。

## 解决方案

### 使用备用布局-使用限定符

* 尺寸限定符
* 使用最小宽度限定符
* 布局别名
* 屏幕方向限定符

**最小宽度限定符： **

通过将屏幕尺寸描述为密度无关像素的度量值， Android 允许创建转为具体的屏幕尺寸而设计的布局。

### 创建可拉抻的九宫格位图

九宫格位图接你上是一种标准的 png 文件，但带有额外的 1 像素边框。

### 布局选择

* 线性布局（LinearLayout)
* 相对布局（RelativeLayout）
* 帧布局（FrameLayout）
* 绝对布局（AbsoluteLayout）
* 约束布局（ConstraintLayout）

### 使用自适应尺寸

* wrap_content
* match_parent
* weight
* dp

不要使用 px

### 百分比适配

1. 以某一分辨率为基准，生成所有分辨率对应像素数列表
2. 将生成像素数列表存放在 res 目录下对应的 value 文件下
3. 根据 UI 设计师给出设计图的尺寸，找到对应像素单位，然后给控件设计就可以

### 使用约束布局

ConstraintLayout

### 今日头条适配方案

`px 值 = dp 值 * metrics.density`  这里的 `density` 是手机的屏幕密度，由系统提供。不同的手机的 `density` 不同，所以我们不能直接使用系统的。