title: 开发问题汇总
---

### 常见问题

#### 相关文章

- [qiankun官方-常见问题](https://qiankun.umijs.org/zh/faq)
- [qiankun 微前端方案实践及总结（一）](https://juejin.cn/post/6844904185910018062#heading-20)
- [qiankun 微前端实践实践及总结（二）](https://juejin.cn/post/6856569463950639117#heading-32)
- [从0实现一个single-spa的前端微服务（上）](https://juejin.cn/post/6844904046822686733)
- [从0实现一个single-spa的前端微服务（中）](https://juejin.cn/post/6844904048043229192)
- [从0实现一个single-spa的前端微服务（下）](https://juejin.cn/post/6844904085200601102#heading-6)
- [基于 qiankun 的微前端最佳实践（图文并茂） - 应用间通信篇](https://juejin.cn/post/6844904151231496200#heading-2)

### 团队开发遇到的其他问题

#### 1. ant-design-vue 组件挂载到父应用问题
**在使用ant-design-vue开发时，dialog,drawer,modal,dropdown,message等组件挂载到body节点导致父子应用样式污染或子应用样式丢失：**

解决方案：

| 组件	          | 参数	                  | 说明	                      | 类型	                     | 默认值	                | 备注	    |
|--------------|----------------------|--------------------------|-------------------------|---------------------|--------|
| drawer       | getContainer         | 指定 Drawer 挂载的 HTML 节点    | HTMLElement             | () => HTMLElement   | 'body' |
| modal        | getContainer         | 指定 Modal 挂载的 HTML 节点     | (instance): HTMLElement | () => document.body |        |
| message      | getContainer         | 配置渲染节点的输出位置              | () => HTMLElement       | () => document.body |        |
| notification | getContainer         | 配置渲染节点的输出位置              | () => HTMLNode          | () => document.body |        |
| dropdown     | getPopupContainer    | 菜单渲染父节点。默认渲染到 body 上，    | Function(triggerNode)   | () => document.body |        |
| cascader     | getPopupContainer    | 菜单渲染父节点。默认渲染到 body 上，    | Function(triggerNode)   | () => document.body |        |
| date-picker  | getCalendarContainer | 定义浮层的容器，默认为 body 上新建 div | function(trigger)       | 无                   |        |
| mentions     | getPopupContainer    | 指定建议框挂载的 HTML 节点         | () => HTMLElement       |                     |        |
| select       | getPopupContainer    | 菜单渲染父节点。默认渲染到 body 上，    | Function(triggerNode)   | () => document.body |        |

#### 2. 微应用页面显示双滚动条的问题
**我们期望微应用页面不出现滚动条，但是由于@ant-design-vue/pro-layout未开放高度属性定制，并且将布局高度设置为100vh的缘故。所以出现了双滚动条。**

解决方案： 覆盖原插件的样式。推荐新建一个css文件<如: layout.css>直接在全局应用：

````css
.ant-layout {
    // 继承父应用的高度减去顶栏和tab高度
    height: calc(100% - 182px);
}

.sidemenu {
    min-height: 100%;
}

.ant-pro-sider-menu-sider.fix-sider-bar .ant-menu-root {
    // 继承父应用的高度减去顶栏高度
    height: calc(100% - 64px);
}

.ant-pro-sider-menu-sider {
    min-height: 100%;
}
````

#### 3. 子应用sider覆盖了父应用的sider
解决方案： 将子应用的sider定位改为sticky。推荐新建一个css文件<如: sider.css>直接在全局应用：

```css
.ant-pro-sider-menu-sider.fix-sider-bar {
  position: sticky;
} 
```

#### 4. 用户显示屏缩放比的兼容
在小屏上缩放比太大会导致页面元素太大，留给子应用的区域太小。

解决方案： 获取浏览器缩放比，动态设置页面zoom。(父应用设置即可)

// utils.js
```js
/**
 * 获取浏览器缩放比
 * */
export const detectZoom = () => {
  let ratio = 0
  const screen = window.screen
  const ua = navigator.userAgent.toLowerCase()
  if (window.devicePixelRatio) {
    ratio = window.devicePixelRatio
  } else if (~ua.indexOf('msie')) {
    if (screen.deviceXDPI && screen.logicalXDPI) {
      ratio = screen.deviceXDPI / screen.logicalXDPI
    }
  } else if (window.outerWidth && window.innerWidth) {
    ratio = window.outerWidth / window.innerWidth
  }

  return ratio
}
```

// layout.vue
```vue
<script>
export default {
  mounted() {
    this.zoom = detectZoom() === 1 ? 1.5 : 1.25
    document.body.style.zoom = this.zoom / 1.5
  }
}
</script>
```
