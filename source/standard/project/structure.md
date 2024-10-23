title: 项目代码目录结构
---

```markdown
── project-name
   ├── .husky                       Git 提交规范文件
   ├── deploy                       部署nginx相关设置
   ├── public                       项目公共目录
   │   ├── favicon.ico                  Favourite 图标
   │   └── index.html                   页面入口 HTML 模板
   ├── src                          项目源码目录
   │   ├── api                          把所有请求数据的接口，按照模块在单独的JS文件中
   │   │   ├── home.js                      首页接口
   │   │   ├── detail.js                    详情页接口，等等
   │   │   ···
   │   ├── assets                       静态资源目录，公共的静态资源，图片，字体等
   │   │   ├── icons                        字体资源
   │   │   ├── images                       图片资源
   │   │   ···
   │   ├── components                   公共组件目录
   │   │   ├── confirm                      弹框组件
   │   │   │   └── index.vue
   │   │   ├── scroll                       局部内容滚动组件
   │   │   │   └── index.vue
   │   │   ···
   |   ├── config                       项目全局配置信息
   |   |   ├── defaultSetting.js            ant design vue 项目默认配置项
   |   |   ├── router.config.js             项目路由配置信息
   |   |—— core                         全局指令，插件，缓存等
   |   |   ├── directives                   自定义指令文件目录
   |   |       ├── action.js                    Action 权限指令
   |   |   ├── bootstrap.js                 执行项目启动时必须要用到的配置信息及缓存
   |   |   ├── icons.js                     统一管理的自定义图标加载表
   |   |   ├── lay_use.js                   Ant Design Vue 组件懒加载，以及定义的全局组件注册
   |   |   ├── use.js                       Vue插件注册入口文件
   |   ├── data                         常量，全局数据
   |   │   ├── address.js                   省市区全量数据
   |   ├── layouts                      布局组件目录
   |   │   ├── BasicLayout.less             基础布局样式文件
   |   │   ├── BasicLayout.vue              基础布局组件
   |   │   ├── BlankLayout.vue              备选的自定义布局组件，空文件
   |   │   ├── index.js                     布局组件整体module输出文件
   |   │   ├── PageView.vue                 全局封装的page-header-wrapper组件，包含面包屑等元素
   |   │   ├── RouteView.vue                全局封装的路由跳转组件，可进行页面缓存及multitab（multitab是项目中自定义的顶部导航组件）的设置
   |   │   ├── UserLayout.vue               用户登录页面的布局组件
   |   │   ...
   │   ├── locales                      国际化语言配置
   |   │   ├── lang                         语言文件列表
   |   │       ├── en-US.js                     英文
   |   │       ├── zh-CN.js                     中文
   |   │   ├── index.js                     整体module输出文件
   │   │   ···
   │   ├── mixins                       公共的mixin混合文件
   │   │   ├── BreadMixin.js                面包屑混入，通过约定路由参数breadcrumbName自定义面包屑末尾名称
   │   │   ├── TableMixin.js                表格页面混入，定义通用的增删改查等操作，与FormMixin.js共同使用快速建立页面模板，与FormMixin.js文件存在一定程度的耦合
   │   │   ├── FormMixin.js                 表单面混入，定义通用的表单操作
   │   │   ···
   │   ├── mock                         mock文件目录，通过环境变量process.env.VUE_APP_PREVIEW开启
   │   │   ···
   │   ├── router                       路由配置目录
   │   │   ├── generator-routers.js         通过用户角色和权限动态添加，删除菜单及路由信息
   │   │   ···
   │   ├── store                        数据中心
   │   │   ├── modules                      store模块目录
   │   │   │   └── app.js                       app全局配置数据
   │   │   │   └── user.js                      用户数据
   │   │   │   ...
   │   │   ├── app-mixin.js                 app全局配置数据混入，主要是geely-design的布局信息
   │   │   ├── device-mixin.js              设备信息数据混入
   │   │   ├── i18n-mixin.js                国际化信息数据混入
   │   │   ├── mutation-types.js            定义 Mutations 的类型
   │   │   ├── getters.js                   获取数据的方法定义
   │   │   └── index.js                     数据中心入口文件
   │   │   ...
   │   ├── style                        全局样式目录
   │   │   ├── global.less                  公共样式信息
   │   │   ···
   │   ├── utils                        通用脚本文件
   │   │   ├── axios                        axios文件封装
   │   │   ···
   │   ├── views                        页面目录，所有业务逻辑的页面都应该放这里
   │   │   ├── home                         应用首页
   │   │   │   └── index.vue
   │   │   ···
   │   ├── main.js                      入口js文件
   │   ├── App.vue                      根组件
   │   └── permission.js                权限判断及路由拦截器
   ├── .browserslistrc               浏览器相关设置
   ├── .editorconfig                 编辑器相关设置
   ├── .env.development              Vue 开发环境的配置
   ├── .env.production               Vue 生成环境的配置
   ├── .eslintignore                 Eslint 配置忽略文件
   ├── .eslintrc.js                  Eslint 配置文件
   ├── .gitignore                    Git 忽略文件
   ├── .prettierignore               prettier 忽略文件
   ├── .prettierrc                   prettier 配置文件
   ├── Dockerfile                    Docker 配置文件
   ├── Jenkinsfile                   Jenkins 配置文件
   ├── LICENSE                       LICENSE
   ├── README.md                     README.md
   ├── babel.config.js               Babel 配置文件
   ├── commitlint.config.js          Git 提交commit钩子类型设置
   ├── jsconfig.json                 JS 功能配置文件
   ├── package.json                  包依赖文件
   └── vue.config.js                 vue/cli 项目脚手架配置
```
