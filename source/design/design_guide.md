title: UI设计规范
---

### 前言

UED设计涵盖布局，字体，颜色，组件，图表图片，交互形式等。其目的是给出设计人员一定的约束，从而形成特定的网站风格。属于互联网常见标准。作为前端开发者，我们也需要有一定的UE/UI设计素养，甚至可以自己设计自己的组件库，制定自己的设计规范。另外，推荐一个在线设计网站，[MasterGo](https://mastergo.com/)。

中后台系统基本上风格趋于一致，格局和模板市场上种类繁多，多不胜数。但是不能做到百分百覆盖，因此在具体开发自己业务和组件时仍然要遵循一定的标准。因此，我总结了本规范，希望能帮助到大家。

### 设计原则

1. 一致性：确保界面风格和交互模式保持一致，提高用户的学习效率。
2. 易用性：保持操作简单明了，降低用户的认知成本。
3. 可扩展性：设计时考虑系统的可扩展性，方便未来进行功能迭代和升级。

### 布局

中后台系统以业务功能为主，因此要尽可能突出业务数据，以操作数据为高优先级，呈现出清晰的层次关系和操作逻辑。由于我们使用了统一脚手架，layout为内置组件，因此开发者重点关注右侧内容区。

建议使用栅格系统把视觉界面区块化，将页面上的信息进行分类和组合，以减少界面的复杂性，同时易于设将来的拓展性。

1. 页面布局：采用左侧导航+顶部辅助工具栏+右侧主工作区的布局方式。
2. 栅格系统：采用24栅格布局，确保页面元素的对齐和间距。
3. 分辨率：设计稿基于1920x1080分辨率，同时适应其他常见分辨率。

#### 首页及图表面板

### 字体规范

1. 主字体：推荐使用雅黑、苹方等易读性强的字体。
2. 字号：标题、正文、辅助信息分别使用不同的字号，保持层次分明。
3. 行高：根据字号调整合适的行高，确保良好的阅读体验。

### 颜色规范

1. 主色调：选择一种主色调，贯穿整个系统，用于突出重要信息和操作。
2. 辅助色：选择2-3种辅助色，用于区分信息层级和视觉引导。
3. 背景色：选择一种淡色背景，保持整体界面的清爽简洁。

系列色值延续ant-design-vue，如下：

<div style="width: 50px; height: 20px; background-color: #1890ff"></div>
<div style="width: 50px; height: 20px; background-color: #722ed1"></div>
<div style="width: 50px; height: 20px; background-color: #13c2c2"></div>
<div style="width: 50px; height: 20px; background-color: #52c41a"></div>
<div style="width: 50px; height: 20px; background-color: #eb2f96"></div>
<div style="width: 50px; height: 20px; background-color: #f5222d"></div>
<div style="width: 50px; height: 20px; background-color: #fa8c16"></div>
<div style="width: 50px; height: 20px; background-color: #fadb14"></div>
<div style="width: 50px; height: 20px; background-color: #fa541c"></div>
<div style="width: 50px; height: 20px; background-color: #2f54eb"></div>
<div style="width: 50px; height: 20px; background-color: #a0d911"></div>
<div style="width: 50px; height: 20px; background-color: #faad14"></div>

### 视觉风格

全局主题色为 `#1890ff`。

### 交互规范

1. 交互反馈：为用户操作提供明确的交互反馈，如按钮点击态、hover效果等。
2. 动画：使用合适的动画效果，提高用户的操作体验，但避免过度动画。
3. 加载提示：在必要时提供加载提示，减轻用户的等待焦虑。

### 数据可视化

我们推荐数据可视化ui库统一使用echarts，数据报表面板自定义推荐使用dataEase。

### 自定义组件及可扩展模块

我们采用了ant-design-vue作为团队统一组件库，因此自定义组件或交互要符合ant-design-vue主题风格。注意按照前面几点来设计。

### 自定义主题及更改默认样式

虽然我们采用ant-design-vue作为统一UI库，但是开发者如果确实有更改默认样式或想要[自定义主题](https://1x.antdv.com/docs/vue/customize-theme-cn/)，可以覆盖样式来达到目的。

可以在官方仓库[这里](https://github.com/vueComponent/ant-design-vue/blob/master/components/style/themes/default.less)来找到自己想要的变量来全局更改样式或自定义主题。
