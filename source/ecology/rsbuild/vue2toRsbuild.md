title: vue/cli项目迁移到Rsbuild
---

公司的项目一直在膨胀，随着功能模块的增加，项目的性能表现和构建编译速度成为项目开发的瓶颈。为了对大型的历史项目进行优化，我进行了一些优化工作。
本文针对构建编译方面的优化工作做下记录和分享。

### 一、项目背景

#### 1. 项目架构
公司有相当数量的历史项目是基于vue2和webpack5开发（vue2+and-design-vue+webpack5），使用官方构建工具 `@vue/cli` 进行构建。一个中型项目本地冷启动时间差不多20多秒，线上Jenkins流水线发布时间已达到90秒，大型和超大型项目时间更是超过2分钟。
为了优化构建编译时间，我对市面上的新型构建工具进行了调研。

#### 2. 构建工具调研
现在新型的构建工具包括vite、rspack、turbopack、esbuild、rollup等。

其中vite是利用原生es模块和现代浏览器特性，以esbuild和rollup为基础开发的快速启动的构建工具。
优点是冷启动速度很快，易于配置，支持多种前端框架，高效热更新。缺点是生态不如webpack，本地开发和生产可能不一致。

Turbopack是Vercel团队推出的 JavaScript 打包器，作为 Webpack 的后继者。其目标是提供超高速的开发体验和构建速度。
优点是极快的构建速度，与webpack兼容性好，对使用vercel发布的项目有更好支持。缺点是生态不足，且团队偏向于vercel本身的投入。

Esbuild 是一个用 Go 语言编写的 JavaScript 和 TypeScript 打包工具，以其惊人的构建速度而著称。
优点是构建速度快，简单的api易于使用。缺点是生态方面不如其他构建工具，一些高级复杂特性需要第三方支持。

Rollup 是一个 JavaScript 模块打包器，专注于 ES6 模块。它常用于构建库和应用，以生成高效的、可优化的代码。
优点是构建包更小，灵活的插件系统，更擅长构建js库。缺点是开发体验不如vite或webpack，配置繁琐。

Rspack 是一个由字节跳动开发的 JavaScript 打包工具，旨在提供比 Webpack 更快的构建速度。它的设计目标是兼容 Webpack 的生态系统，同时提升性能。
优点是兼容webpack，更易迁移，用Rust重写webpack，速度快，在大型项目表现更好，技术团队对issue相应快。缺点是刚刚开源，一些插件和文档需要持续完善。

> 以上比较来自各构建工具官网介绍和ai支持。

根据我们部门历史项目中后台管理系统的占比大的性质，结合开发人员素质和对迁移的难度，我选择了Rsbuild作为新的构建工具。

#### 3. Rsbuild介绍
那么Rsbuild与Rspack有什么关系呢？Rspack是一个基于Rust重写webpack的高性能构建工具，而Rsbuild是基于Rspack的有预设配置的开箱即用的构建工具。
可以把Rspack类比webpack，把Rsbuild类比为一个现代化的 Create React App 或 Vue CLI。

[Rsbuild官网](https://rsbuild.dev/zh/index)

> 从Vue/cli迁移：https://rsbuild.dev/zh/guide/migration/vue-cli

### 二、迁移步骤

官方的迁移指南比较简单，针对我们项目的技术栈和特点。整理下比较完善的步骤：

#### 1. node版本

请保证node版本 >= 16，推荐安装nvm来管理各个版本的node。

[node版本管理工具-nvm](https://github.com/nvm-sh/nvm)

#### 2. 基础配置更改

1） **依赖替换**

移除之前的vue/cli脚手架相关的依赖、core-js依赖和babel相关依赖。安装Rsbuild的相关依赖和插件。

移除 Vue CLI 的依赖：
```shell
pnpm remove @vue/cli-service @vue/cli-plugin-babel @vue/cli-plugin-eslint core-js
```
安装 Rsbuild 的依赖：
```shell
pnpm add @rsbuild/core @rsbuild/plugin-vue -D
```

> 注意：vue2的版本至少要升级到2.7。[在已有项目中使用-vue2](https://rsbuild.dev/zh/guide/framework/vue#%E5%9C%A8%E5%B7%B2%E6%9C%89%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-vue-2)

最后的依赖配置如下：

```json
{
  "dependencies" : {
    "vue": "2.7.16"
  },
  "devDependencies": {
    "@rsbuild/core": "^1.1.13",
    "@rsbuild/plugin-babel": "^1.0.3",
    "@rsbuild/plugin-less": "^1.1.0",
    "@rsbuild/plugin-vue2": "^1.0.2",
    "@rsbuild/plugin-vue2-jsx": "^1.0.3",
    "vue-template-compiler": "2.7.16"
  }
}
```

2） **启动脚本**

之前的 `vue-cli-serve` 命令改为Rsbuild的CLI命令： `rsbuild`。

> 由于Rsbuild 未集成 ESLint，因此没有提供用于替换 `vue-cli-service lint` 的命令，你可以直接使用 ESLint 的 [CLI 命令](https://eslint.org/docs/latest/use/command-line-interface) 作为替代。

```json
{
  "scripts": {
-    "serve": "vue-cli-service serve",
-    "build:sit": "vue-cli-service build --mode sit",
-    "build": "vue-cli-service build",
-    "lint": "vue-cli-service lint",
-    "report": "vue-cli-service build --report",
+    "serve": "rsbuild dev",
+    "build:sit": "rsbuild build --env-mode sit",
+    "build": "rsbuild build",
+    "lint": "npx eslint lint",
+    "report": "cross-env BUNDLE_ANALYZE=true rsbuild build --report"
  }
}
```

3） **创建rsbuild.config文件**

在项目根目录下创建Rsbuild的配置文件 `rsbuild.config.js`

```js
import { defineConfig } from '@rsbuild/core';
import { pluginVue } from '@rsbuild/plugin-vue';

export default defineConfig({
  plugins: [pluginVue()],
  source: {
    // 指定入口文件
    entry: {
      index: './src/main.js',
    },
  },
});
```

> 这个vue3的版本的，我们的老项目是vue2的，因此vue插件应该使用 `import { pluginVue2 } from '@rsbuild/plugin-vue2'`。

4） **HTML更改**

将 `public/index.html` 文件更改并作为入口文件模板。

```html
-  <link rel="icon" href="<%= BASE_URL %>favicon.ico">
+  <link rel="icon" href="<%= assetPrefix %>/favicon.ico">
```

> 注意 `<%= assetPrefix %>/` 最后有个斜杠 "/"。

rsbuild.config更改模板：

```js
export default defineConfig({
  html: {
    template: './public/index.html',
  },
});
```

5） **环境变量统一配置**

Rsbuild的环境变量默认注入 `PUBLIC_`, 针对vue2的老项目，要统一更改。

```js
import { defineConfig, loadEnv } from '@rsbuild/core';

const { publicVars } = loadEnv({ prefixes: ['VUE_APP_'] });

export default defineConfig({
  source: {
    define: publicVars,
  },
});
```

之后在代码里如果想要使用之前的全局变量可直接使用 `process.env` 或 `import.meta.env`。

如果将publicVars打印，我们可以得到：

```js
{                                                  
  'import.meta.env.VUE_APP_API_BASE_URL': '"/api"',
  'process.env.VUE_APP_API_BASE_URL': '"/api"',    
  'import.meta.env.VUE_APP_ENVIRONMENT': '"dev"',  
  'process.env.VUE_APP_ENVIRONMENT': '"dev"',      
  'import.meta.env.VUE_APP_PREVIEW': '"true"',     
  'process.env.VUE_APP_PREVIEW': '"true"',         
  'import.meta.env.VUE_APP_SENTRY_DSN': '""',
  'process.env.VUE_APP_SENTRY_DSN': '""',
  'import.meta.env.VUE_APP_SENTRY_SITE_DOMAIN': '""',
  'process.env.VUE_APP_SENTRY_SITE_DOMAIN': '""',
  'import.meta.env.VUE_APP_SERVICE_PROXY': '"http://api-ts.com/v1"',
  'process.env.VUE_APP_SERVICE_PROXY': '"http://api-ts.com/v1"'
}
```

> 注意：上面的有一些变量是空字符串，即使一些变量只在生产环境使用也要在.env文件配置空变量，否则会报错，如：`VUE_APP_SENTRY_DSN`，只在生产环境使用sentry，但是也要在.env文件配置空变量。

6） 删除多余文件

将之前的 `vue.config.js` 和 `babel.config.js` 删除。

#### 3. webpack插件兼容

由于 Rsbuild 内置了一些常见的 loader 和 plugin，所以你可以移除以下依赖和相关的配置，这会显著提升项目的依赖安装速度。

- css-loader
- babel-loader
- style-loader
- postcss-loader
- html-webpack-plugin
- mini-css-extract-plugin
- autoprefixer
- @babel/core
- @babel/preset-env
- @babel/preset-typescript
- @babel/runtime

另外Rsbuild官方提供了一些插件，可以直接替换webpack的插件：[Rsbuild 插件](https://rsbuild.dev/zh/plugins/list/index)

#### 4. 其他优化

1）**less兼容**

上面已经安装了 `@rsbuild/plugin-less` ，在plugins加入pluginLess即可。但是因为Rsbuild内置了lessV4版本，所以版本不一致会报错。
注意：如果你的项目less版本小于4。需要配置自己的less实际安装版本。用implementation指定。

```js
import { defineConfig, loadEnv } from '@rsbuild/core';
import { pluginLess } from '@rsbuild/plugin-less'

export default defineConfig({
    plugins: [
        pluginLess({
            lessLoaderOptions: {
                implementation: require('less')
            }
        })
    ],
});
```

2）**样式穿透**

将全局的样式穿透 `/deep/` 改为 `::v-deep`，确保在scoped的style标签下。

3） **开发模式**

按着之前的vue2项目的.env配置多环境打包，在Rsbuild下可以使用 `envMode` 参数来区分各环境。

```json
{
  "scripts": {
    "test": "rsbuild build --env-mode test"
  }
}
```

在编译命令指定当前模式，在rsbuild.config.js中可直接通过 `envMode` 字段来确定当前模式，可进一步对各个环境编译做区分。

4） **jsx/babel**

```js
import { defineConfig } from '@rsbuild/core'
import { pluginVue2 } from '@rsbuild/plugin-vue2'
import { pluginVue2Jsx } from '@rsbuild/plugin-vue2-jsx'
import { pluginBabel } from '@rsbuild/plugin-babel'

export default defineConfig({
    plugins: [
        pluginVue2(),
        pluginVue2Jsx(),
        pluginBabel()
    ]
})
```

5） **删除console打印**

在performance字段下可以直接配置。

```js
import { defineConfig } from '@rsbuild/core'

export default defineConfig({
    performance: {
        removeConsole: true,
    },
})
```

6） **移除moment语言包**

和删除console一样，在performance字段下可以直接配置。

```js
import { defineConfig } from '@rsbuild/core'

export default defineConfig({
    performance: {
        removeMomentLocale: true,
    },
})
```

7） **按需引入UI库**

```js
import { defineConfig } from '@rsbuild/core'

export default defineConfig({
    source: {
        transformImport: [
            {
                libraryName: 'ant-design-vue',
                libraryDirectory: 'es',
                style: true,
            },
        ],
    },
})
```

8） 生产环境cdn

对生产环境的一些依赖可以使用cdn，配置如下：

```js
import { defineConfig, loadEnv } from '@rsbuild/core'

const assetsCDN = {
    // rspack build externals
    externals: {
        vue: 'Vue',
        'vue-router': 'VueRouter',
        vuex: 'Vuex',
        axios: 'axios',
    },
    css: [],
    // https://unpkg.com/browse/vue@2.6.10/
    js: [
        '//cdnjs.h3c.com/npm/vue@2.7.16/dist/vue.min.js',
        '//cdn.jsdelivr.net/npm/vue-router@3.6.5/dist/vue-router.min.js',
        '//cdn.jsdelivr.net/npm/vuex@4.1.0/dist/vuex.global.min.js',
        '//cdn.jsdelivr.net/npm/axios@1.7.9/dist/axios.min.js',
    ],
}

export default defineConfig({
    html: {
        template: './public/index.html',
        templateParameters: {
            title: 'title',
            cdn: assetsCDN,
        },
    },
})
```

templateParameters可定义模板中的参数我们增加cdn字段和title字段，此时的html也要更改，`htmlWebpackPlugin.options` 直接删掉，更改后如下：

```html
- <title><%= htmlWebpackPlugin.options.title %></title>
+ <title><%= title %></title>

<!-- require cdn assets css -->
- <% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.css) { %>
- <link rel="stylesheet" href="<%= htmlWebpackPlugin.options.cdn.css[i] %>" />
+ <% for (var i in cdn && cdn.css) { %>
+ <link rel="stylesheet" href="<%= cdn.css[i] %>" />
<% } %>

<!-- require cdn assets js -->
- <% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.js) { %>
- <script type="text/javascript" src="<%= htmlWebpackPlugin.options.cdn.js[i] %>"></script>
+ <% for (var i in cdn && cdn.js) { %>
+ <script type="text/javascript" src="<%= cdn.js[i] %>"></script>
<% } %>
<!-- built files will be auto injected -->
```

> 更多优化可参考官网: [提升构建性能](https://rsbuild.dev/zh/guide/optimization/build-performance)

#### 5. 完整的rsbuild.config配置

```js
import { defineConfig, loadEnv } from '@rsbuild/core'
import { pluginVue2 } from '@rsbuild/plugin-vue2'
import { pluginVue2Jsx } from '@rsbuild/plugin-vue2-jsx'
import { pluginBabel } from '@rsbuild/plugin-babel'
import { pluginLess } from '@rsbuild/plugin-less'

const isProd = process.env.NODE_ENV === 'production'

const { publicVars } = loadEnv({ prefixes: ['VUE_APP_'] })

const assetsCDN = {
    // rspack build externals
    externals: {
        vue: 'Vue',
        'vue-router': 'VueRouter',
        vuex: 'Vuex',
        axios: 'axios',
    },
    css: [],
    // https://unpkg.com/browse/vue@2.6.10/
    js: [
        '//cdnjs.h3c.com/npm/vue@2.7.16/dist/vue.min.js',
        '//cdn.jsdelivr.net/npm/vue-router@3.6.5/dist/vue-router.min.js',
        '//cdn.jsdelivr.net/npm/vuex@4.1.0/dist/vuex.global.min.js',
        '//cdn.jsdelivr.net/npm/axios@1.7.9/dist/axios.min.js',
    ],
}

export default defineConfig({
    plugins: [
        pluginVue2(),
        pluginVue2Jsx(),
        pluginBabel(),
        pluginLess({
            lessLoaderOptions: {
                lessOptions: {
                    modifyVars: {
                        // less vars，customize ant design theme
                        'primary-color': '#2a50d7',
                        'success-color': '#22835a',
                        'warning-color': '#e37318',
                        'error-color': '#ca483e',
                        'border-radius-base': '4px',
                        'box-shadow-base': '0 8px 16px rgba(0, 0, 0, 0.16)',
                    },
                    // DO NOT REMOVE THIS LINE
                    javascriptEnabled: true,
                },
                implementation: require('less'),
            },
        }),
    ],
    html: {
        template: './public/index.html',
        templateParameters: {
            title: 'title',
            cdn: assetsCDN,
        },
    },
    source: {
        entry: {
            index: './src/main.js',
        },
        define: {
            ...publicVars,
        },
        transformImport: [
            {
                libraryName: 'ant-design-vue',
                libraryDirectory: 'es',
                style: true,
            },
        ],
    },
    output: {
        externals: isProd ? assetsCDN.externals : {},
        polyfill: 'usage',
    },
    resolve: {
        alias: {
            '@': './src',
            '@util': './src/util',
        },
    },
    tools: {
        bundlerChain: (chain, { CHAIN_ID }) => {
            chain.module.rule(CHAIN_ID.RULE.SVG).oneOfs.clear()
            chain.module.rule(CHAIN_ID.RULE.SVG).use('vue-svg-loader').loader('vue-svg-loader')
        },
    },
    server: {
        // development server port 8080
        port: 8080,
        // If you want to turn on the proxy, please remove the mockjs /src/main.jsL11
        proxy: {
            '/api': {
                target: process.env.VUE_APP_SERVICE_PROXY,
                changeOrigin: true,
                pathRewrite: {
                    // 表示需要rewrite重写的
                    '^/api': '',
                },
            },
        },
    },
    performance: {
        chunkSplit: {
            strategy: 'split-by-experience',
        },
        removeConsole: isProd ? true : undefined,
        removeMomentLocale: true,
        // analyze功能，安装cross-env并启用下面配置
        // bundleAnalyze: {},
    },
})
```

### 三、其他问题

官方针对一些常用问题已经做了汇总，遇到问题可先看下：[常见问题](https://rsbuild.dev/zh/guide/faq/exceptions)

- 图片通过require引入失效？

试着改为import。

- Failed to compile, check the errors for troubleshooting.

这是同一个插件在 `plugins` 和 `tools.rspack` 同时更改造成冲突了，只保留一个地方的更改。

### 参考
- [前端构建工具对比](https://www.cnblogs.com/94pm/p/18542399)
- [Rsbuild 1.0 发布：专为 Rspack 打造的构建工具](https://juejin.cn/post/7412546558845747251)
- [一个关于打包工具的故事：从 webpack 迁移到 Rspack](https://juejin.cn/post/7425598941859676170)
