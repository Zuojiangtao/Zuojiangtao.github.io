title: 移动端适配方案
---

### 一、前言

移动端适配是前端开发中很重要的一块，无论是项目开发还是面试都会涉及到。

网上已经有很多详细解决方案了，正好公司需要定制移动端脚手架，借此机会将这几种方案重新梳理下。

开源的移动端脚手架：[基于vue-cli5及vw方案实现的移动端企业级工程架构](https://github.com/Zuojiangtao/vue-vw-mobile)

### 二、一些简单概念

#### 1.像素

像素（pixel，缩写为 px）是计算机图形和数字图像领域中的基本单位，用于表示显示设备上的一个点或一个图像元素。

在我们前端样式设置时，像素一般使用css像素作为单位，但是实际上对应到设备上，具体负责显示元素的是**物理像素**。

物理像素指的是显示器最小可视显示单元，一个个小的单位物理像素各自显示不同的颜色和亮度来组合成一个完整的物体。

设备像素决定物体显示效果，但是是由系统来控制设备的显示的，我们称之为**设备独立像素**。不同的设备独立像素不一样，高清显示屏的独立像素比较大，能够显示更高清的图片。
但是，如果将普通图片放到高清显示屏上，图片会变模糊，所以需要我们在不同的设备上显示不同倍率的图片，如UI常说的2倍图，3倍图。

css像素在不同的设备独立像素显示效果不同，如1px的点在1倍屏显示是1x1的设备像素显示，在2倍屏上就变成2x2的是设备像素显示，在3倍屏上则是3x3的设备像素显示。

[//]: # (这里是简单的一些概念，如果需要了解更多，请看另一篇文章：[]&#40;&#41;。)

#### 2.相对单位

目前，为不同设备分辨率适配，css由多种相对单位，这里我们简要介绍rem和viewport。

rem，是相对HTML的根字体大小的相对单位，如某项目html根字体为20px，那么1rem等于20px。通过动态设置根字体大小，我们可适配不同分辨率的设备。

vw，是浏览器可视区域的视口（viewport），100vw等于视口的宽度。在移动端要注意，布局视口和视觉视口不一样，要通过meta标签设置设备宽度为视口宽度。`<meta name="viewport" content="width=device-width, initial-scale=1.0">` 。

### 三、几种方案对比

#### 1.百分比和flex布局

这种方案其实是对某些打开的区块做整体性布局适配比较合适，针对细化的内容块、高度以及字体的适配不是很好。

```css
.logo {
    width: 20%;
    height: 17.5%;
    font-size: 20px;
}
```

此方案缺陷较大，不建议使用。

#### 2.css属性@media

```css
@media only screen and(min-width: 375px) {
    .container {
        width: 100px;
    }
}

@media only screen and(min-width: 500px) {
    .container {
        width: 175px;
    }
}

@media only screen and(min-width: 750px) {
    .container {
        width: 200px;
    }
}
```

优点：

- 更细致、更准确还原视觉稿设计
- 对特殊样式可单独处理

缺点：

- 工程量大
- 不易维护

#### 3.使用第三方css库

一些优秀的css库可直接使用，像bootstrap、UnoCSS和tailwind都是优秀的css库，适用于h5活动页、更新频率低的网站。 通过原子化的样式类名直接控制内容区块在不同宽度设备的显示。

```html
// tailwindcss
<div class="container mx-auto"></div>

<div class="w-full h-[50px] flex flex-row bg-white"></div>
```

优点：
- 对熟悉库的人来说开发效率高，开发体验好
- 可省去自己命名class的烦恼
- ui库自带样式设计，对没有UI的团队友好

缺点：

- 上手有成本，对一些class要记忆，需要一段适应时间
- 维护成本高
- 更改样式成本高

#### 4.rem/flexible方案

rem是相对单位，可以通过js在html动态设置根字体大小，然后可以直接按照ui视觉稿开发。这种方案适用于对ui没那么严格的项目，尤其是一些B端产品。

可直接安装依赖amfe-flexible使用：
```bash
npm install amfe-flexible -D
```

```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no">
<script src="./node_modules/amfe-flexible/index.js"></script>
```

这种方案兴起于在16，17年。那会较为流行，但是在实际使用过程中发现有一些问题，现在amfe-flexible官方也不推荐这种方案了。

优点：

- 简单快速高效
- 可覆盖绝大部分屏幕的适配，保持等比缩放

缺点：

- 在高清屏上元素尺寸显得大，字体又显得小
- 在某些机型上布局错乱
- 对有小数单位的处理有误差

#### 5.vw

得益于viewport单位得到众多浏览器的兼容，vw移动端适配方案被各大厂团队所采用。绝大多数的项目都可以使用这种方案。

```css
.img {
    width: 10vw; // 1vw = 750px * 1% = 7.5px
}
```

vw是基于视口的相对单位，100vw = device-width，因此可从底层来解决不同尺寸屏幕的适配问题。

优点：

- 不依赖font-size尺寸设置
- 可覆盖rem方案所有优点

缺点：

- 兼容性问题，少数低版本手机系统 ios8、android4.4以下不支持

#### 6.其他

其他方案也有通过动态更改meta标签设置缩放比并配合css缩放属性scale来做适配、rem/vw/px混用适配等方案。我没使用过就不做对比了，感兴趣可根据关键词自行搜索。

### 四、开发指南

#### 1.meta标签

| 字段            | 说明                                                |
|:--------------|:--------------------------------------------------|
| width         | 设置 layout viewport 的宽度，为一个正整数，或字符串 "width-device" |
| height	       | 设置页面的初始缩放值，为一个数字，可以带小数                            |
| initial-scale | 允许用户的最小缩放值，为一个数字，可以带小数                            |
| minimum-scale | 允许用户的最大缩放值，为一个数字，可以带小数                            |
| maximum-scale | 设置 layout viewport 的高度，这个属性对我们并不重要，很少使用           |
| user-scalable | 是否允许用户进行缩放，值为"no"或"yes", no 代表不允许，yes 代表允许        |

移动端通过此标签来控制不允许用户缩放页面，viewport视口宽度为设备宽度。

#### 2.1px问题

通过之前的概念介绍，可知道像素分为设备像素和设备独立像素，css像素的显示在不同屏幕上显示效果不一样。

解决方案：
1. 动态更改像素大小，在普通屏幕下就设置1px，在2倍屏和3倍屏下显示0.5px（因为实测3倍屏在小于0.46px后就无法显示了，所以3倍屏和2倍屏一样）；

2. 使用边框图片来代替

    ```css
    .fix-1px {
        border：1px solid transparent;
        border-image: url('./../../image.jpg') 2 repeat;
    }
    ```

3. 伪元素

    ```css
    .fix-1px {
        content:'';
        display:block;
        position:absolute;
        width: 100%;
        border-top:1px solid red;
        height:0px;
        transform:scale(1, 0.5);
        top:0;
        left:0;
    }
    ```
4. postcss-write-svg插件
5. 其他方案 [7 种方法解决移动端 Retina 屏幕 1px 边框问题](https://blog.csdn.net/u012138854/article/details/112801066)

#### 3.单位换算

一般我们开发通过插件来设置基准值，在写单位时直接按照视觉稿即可。如果采用rem方案使用 `postcss-pxtorem`, 如果采用了vw方案则使用 `postcss-px-to-viewport`。

这里我以 `postcss-px-to-viewport` 为例来演示换算插件的使用流程：

1. 安装插件

    ```bash
    npm install postcss-px-to-viewport -D
    ```

2. 配置 .postcssrc.js 文件

    ```js
    module.exports = {
        // ...
        plugins: {
            "postcss-px-to-viewport": {
                viewportWidth: 750,     // (Number) The width of the viewport.
                viewportHeight: 1334,    // (Number) The height of the viewport.
                unitPrecision: 3,       // (Number) The decimal numbers to allow the REM units to grow to.
                viewportUnit: 'vw',     // (String) Expected units.
                selectorBlackList: ['.ignore', '.hairlines'],  // (Array) The selectors to ignore and leave as px.
                minPixelValue: 1,       // (Number) Set the minimum pixel value to replace.
                mediaQuery: false       // (Boolean) Allow px to be converted in media queries.
            },
        },
        // ...
    }
    ```
   
3. 设置单位

    ```css
    .div-vw {
        width: 120px;
        text-align: center;
    }
    ```

#### 4.针对不同dpr屏幕图片尺寸

对于图片我们希望在不同设备像素的屏幕上显示对应的倍率图片，以追求最好的体验与流畅度。

解决方案：
1. 利用css媒体查询属性

    less:
    ```less
    // ios
    .bg-image(@url) {
      background-image: ~"url('~@/assets/images/2x/@{url}@2x.png')";
      @media (-webkit-min-device-pixel-ratio: 3), (min-device-pixel-ratio: 3) {
        background-image: ~"url('~@/assets/images/3x/@{url}@3x.png')";
      }
      background-size: 100% 100%;
    }
    // android
    .bg-image(@url) {
      background-image: ~"url('~@/assets/images/xhdpi/@{url}.png')";
      @media (-webkit-min-device-pixel-ratio: 3), (min-device-pixel-ratio: 3) {
        background-image: ~"url('~@/assets/images/xxhdpi/@{url}.png')";
      }
      background-size: 100% 100%;
    }
    ```

2. 使用img的srcset属性

    ```html
    <img src="./img.png" srcset="image@2x.png 2x,image@3x.png 3x">
    ```

#### 5.基准值解释

前面设置postcss插件中viewportWidth字段，对应的是我们视觉稿的宽度。

### 五、参考资料

- [百度百科-像素](https://baike.baidu.com/item/%E5%83%8F%E7%B4%A0/95084?fr=aladdin#2_1)
- [tailwindcss官网](https://www.tailwindcss.cn/)
- [前端开发移动端的适配方案推荐-大漠](https://www.zhihu.com/question/507570055)
