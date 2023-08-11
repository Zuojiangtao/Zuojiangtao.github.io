title: vite项目优化插件汇总
---

前端项目性能优化和开发体验优化是无法避免的一个话题，在之前使用webpack做应用构建的时候，我们有许多手段和插件来做优化。当我们使用vite构建应用时也有许多插件来做优化，本文对vite的优化插件或方法做下汇总。

### 一、性能优化

#### 1. html处理

[vite-plugin-html](https://github.com/vbenjs/vite-plugin-html)

vite-plugin-html 是一个 Vite 插件，用于将传统 HTML 文件作为输出文件使用。该插件基于 html-webpack-plugin 插件创建，但是是专门为 Vite 构建的优化工具，不依赖于 webpack。

安装：
```shell
pnpm add vite-plugin-html -D
```

使用：

```js
import { defineConfig, Plugin } from 'vite'
import vue from '@vitejs/plugin-vue'

import { createHtmlPlugin } from 'vite-plugin-html'

export default defineConfig({
  plugins: [
    vue(),
    createHtmlPlugin({
      minify: true,
      /**
       * 在这里写entry后，你将不需要在`index.html`内添加 script 标签，原有标签需要删除
       * @default src/main.ts
       */
      entry: 'src/main.ts',
      /**
       * 如果你想将 `index.html`存放在指定文件夹，可以修改它，否则不需要配置
       * @default index.html
       */
      template: 'public/index.html',

      /**
       * 需要注入 index.html ejs 模版的数据
       */
      inject: {
        data: {
          title: 'index',
          injectScript: `<script src="./inject.js"></script>`,
        },
        tags: [
          {
            injectTo: 'body-prepend',
            tag: 'div',
            attrs: {
              id: 'tag',
            },
          },
        ],
      },
    }),
  ],
})
```

#### 2. 图片压缩

[vite-plugin-image-optimizer](https://github.com/FatehAK/vite-plugin-image-optimizer)

vite-plugin-image-optimizer 是一个 Vite 插件，用于自动优化应用程序中的图像文件。该插件可以帮助开发者减少应用程序的加载时间和带宽使用，从而提高用户体验和网站性能。

安装：
```shell
pnpm add vite-plugin-image-optimizer -D
```

使用：
```js
import { ViteImageOptimizer } from 'vite-plugin-image-optimizer';
import { defineConfig } from 'vite';

export default defineConfig(() => {
  return {
    plugins: [
      ViteImageOptimizer({
        /* pass your config */
      }),
    ],
  };
});
```

#### 3. gzip代码压缩

[vite-plugin-compression](https://github.com/anncwb/vite-plugin-compression)

vite-plugin-compression是一个基于Vite的插件,用于gzip或Brotli压缩你的资源，从而减少页面的加载时间和网络带宽，提高用户访问速度和体验。

安装：
```shell
pnpm add vite-plugin-compression -D
```

使用：
```js
import viteCompression from 'vite-plugin-compression';

export default () => {
  return {
    plugins: [viteCompression()],
  };
};
```

#### 4. 依赖cdn

[vite-plugin-external-cdn](https://github.com/Zuojiangtao/vite-plugin-external-cdn)

vite-plugin-external-cdn一款将依赖转换为cdn的vite插件，脱胎于vite-plugin-cdn-import，由于vite-plugin-cdn-import停止维护，我fork了一份到本地，并对一些bug做了修复，做了开源。

安装：
```shell
pnpm add vite-plugin-external-cdn -D
```

使用：
```js
// vite.config.js
import reactRefresh from '@vitejs/plugin-react-refresh'
import transformExternalCDN from 'vite-plugin-external-cdn'

export default {
    plugins: [
      transformExternalCDN({
            modules: [
                {
                    name: 'react',
                    var: 'React',
                    path: `umd/react.production.min.js`,
                },
                {
                    name: 'react-dom',
                    var: 'ReactDOM',
                    path: `umd/react-dom.production.min.js`,
                },
            ],
        }),
    ],
}
```

#### 5. 生产环境移除console

- [vite-plugin-remove-console](https://github.com/xiaoxian521/vite-plugin-remove-console)

vite-plugin-remove-console事一款生产环境删除所有指定console类型的vite插件。

安装：
```shell
pnpm add vite-plugin-remove-console -D
```

使用：
```js
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import removeConsole from "vite-plugin-remove-console";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue(), removeConsole()]
});
```

- 直接vite配置esbuild - drop

```js
esbuild: {
  // 移除日志打印及debugger,可在env配置VITE_DROP_CONSOLE
  drop: VITE_DROP_CONSOLE ? ['console', 'debugger'] : []
}
```

#### 6. 依赖分析

[rollup-plugin-visualizer](https://github.com/btd/rollup-plugin-visualizer)

rollup-plugin-visualizer 是一个用于可视化 Rollup 打包输出文件的插件，它可以帮助开发者分析和优化代码依赖关系，从而提高代码性能和可读性。

安装：
```shell
pnpm add rollup-plugin-visualizer -D
```

使用：
```ts
// vite.config.ts
import vue from "@vitejs/plugin-vue";
import { defineConfig } from 'vite'
import type { PluginOption } from 'vite';
import visualizer from 'rollup-plugin-visualizer';
export default defineConfig({
    plugins: [
        vue(),
        visualizer({
            // template: 'treemap', // sunburst | treemap | network | raw-data | list
            filename: './node_modules/.cache/visualizer/stats.html',
            open: true,
            gzipSize: true,
            brotliSize: true,
        }) as PluginOption
    ]
})
```

### 二、开发体验优化

#### 1. 自动导入第三方ui库组件

[unplugin-vue-components](https://github.com/antfu/unplugin-vue-components)

unplugin-vue-components 是一个 Vite 插件，用于自动化地将您的 Vue.js 组件库导出为独立包。该插件可以帮助您更轻松地编写和共享 Vue 组件，并提供更好的开发体验和代码复用性。

安装：
```shell
pnpm add unplugin-vue-components -D
```

使用：
```ts
import vue from "@vitejs/plugin-vue";
import { defineConfig } from 'vite';
import Components from 'unplugin-vue-components/vite';
import { AntDesignVueResolver } from 'unplugin-vue-components/resolvers';

export default defineConfig({
    plugins: [
        vue(),
        Components({
            resolvers: [
                AntDesignVueResolver({
                    importStyle: 'less',
                    resolveIcons: true,
                }),
            ],
            dts: 'types/components.d.ts',
            include: [/\.vue$/, /\.vue\?vue/, /\.md$/],
            version: 3,
        })
    ]
})
```

#### 2. 自动引入类型定义

[unplugin-auto-import](https://github.com/antfu/unplugin-auto-import)

unplugin-auto-import 是一个 Vite 插件，可以自动将需要的模块或库注入到 JavaScript 或 TypeScript 文件中。该插件可以帮助开发者减少手动导入模块的工作量，并且可以防止由于拼写错误或路径错误导致的编译错误。

安装：
```shell
pnpm add unplugin-auto-import -D
```

使用：
```ts
import vue from "@vitejs/plugin-vue";
import { defineConfig } from 'vite';
import type { PluginOption } from 'vite';
import AutoImport from 'unplugin-auto-import/vite';

export default defineConfig({
    plugins: [
        vue(),
        AutoImport({
            imports: [
                'vue',
                'vue-router',
                {
                    '@vueuse/core': [
                        // named imports
                        'useMouse', // import { useMouse } from '@vueuse/core',
                        'usePreferredDark',
                        'useDark',
                        'useTitle',
                        // alias
                        ['useFetch', 'useMyFetch'], // import { useFetch as useMyFetch } from '@vueuse/core',
                    ],
                    axios: [
                        // default imports
                        ['default', 'axios'], // import { default as axios } from 'axios',
                    ],
                },
            ],
            // resolvers: [AntDesignVueResolver()],
            vueTemplate: true,
            cache: true,
            dirs: ['src/hooks', 'src/components'],
            dts: 'types/auto-imports.d.ts',
        })
    ]
})
```

在vue组件直接使用相关api，不需要手动引入：
```vue
<script>
const count = ref(0)
const doubled = computed(() => count.value * 2)
</script>
```

#### 3. svg作为组件使用

[vite-svg-loader](https://github.com/jpkleemans/vite-svg-loader)

vite-svg-loader 是一个 Vite 插件，用于加载和处理 SVG 文件。它可以帮助开发者在应用程序中无缝使用 SVG 图像，并提供了一些额外的功能来优化和定制 SVG 图像的使用。

安装：
```shell
pnpm add vite-svg-loader -D
```

使用：
```ts
import vue from "@vitejs/plugin-vue";
import { defineConfig } from 'vite';
import type { PluginOption } from 'vite';
import svgLoader from 'vite-svg-loader';

export default defineConfig({
    plugins: [
        vue(),
        svgLoader({
            defaultImport: 'url', // or 'raw'
        })
    ]
})
```

在vue组件使用svg作为组件：
```vue
<template>
  <MyIcon />
</template>

<script lang="ts" setup>
import MyIcon from '@/assets/icons/cancel.svg?component';
</script>
```

#### 4. 在md文档中直接使用vue组件

[vite-plugin-md](https://github.com/antfu/vite-plugin-md)

vite-plugin-md 是一个 Vite 插件，用于在 Vue 项目中加载和渲染 Markdown 文件。它可以帮助开发者更方便地在应用程序中使用和管理 Markdown 内容，将 Markdown 文件转换为可交互的 HTML 页面。

安装：
```shell
pnpm add vite-plugin-md -D
```

使用：
```ts
import vue from "@vitejs/plugin-vue";
import { defineConfig } from 'vite';
import Markdown from 'vite-plugin-md';

export default defineConfig({
    plugins: [
        vue({
            include: [/\.vue$/, /\.md$/], // <--
        }),
        // 为了解决 Error [ERR_PACKAGE_PATH_NOT_EXPORTED], 在package.json增加 "type": "module"
        Markdown(),
    ]
})
```

在ts类型定义：
```ts
// types/global.d.ts
declare module '*.vue' {
  import type { ComponentOptions } from 'vue'
  const Component: ComponentOptions
  export default Component
}

declare module '*.md' {
  import type { ComponentOptions } from 'vue'
  const Component: ComponentOptions
  export default Component
}
```

在md文档中展示vue组件：
```markdown
# My Page
There I was, there I was ... in the jungle. Then I started hearing this ticking sound and I realized it was some sort of _counter_?

<Counter :init='5'/>

I looked a bit closer and realized I could **press** this counter and it would change! What is this magic?
```

在vue组件中使用md文档：
```vue
<template>
  <HelloWorld />
</template>

<script>
import HelloWorld from './README.md'

export default {
  components: {
    HelloWorld,
  },
}
</script>
```

#### 5. 移动端vconsole

[vite-plugin-vconsole](https://github.com/vadxq/vite-plugin-vconsole)

一个适用于 Vite v2+的插件，帮助开发者在各个环境下方便使用VConsole的功能。可以方便配置区分环境，根据环境动态加载VConsole，支持多页面配置。

安装：
```shell
pnpm add vite-plugin-vconsole -D
```

使用：
```js
// tips: 如果引用path提示不存在，可以引入@types/node包
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import { viteVConsole } from 'vite-plugin-vconsole';
import * as path from 'path'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    viteVConsole({
      entry: path.resolve('src/main.ts'), // 或者可以使用这个配置: [path.resolve('src/main.ts')]
      enabled: true, // 可自行结合 mode 和 command 进行判断
      config: {
        maxLogNumber: 1000,
        theme: 'dark'
      }
    })
  ]
});
```

#### 6. mock服务使用

[vite-plugin-mock](https://github.com/vbenjs/vite-plugin-mock)

vite 的数据模拟插件，是基于 vite.js 开发的。 并同时支持本地环境和生产环境。 Connect 服务中间件在本地使用，mockjs 在生产环境中使用。

安装：
```shell
pnpm add mockjs
pnpm add vite-plugin-mock -D
```

使用：

- 开发环境：

```ts
// vite.config.ts 配置
import { UserConfigExport, ConfigEnv } from 'vite'

import { viteMockServe } from 'vite-plugin-mock'
import vue from '@vitejs/plugin-vue'

export default ({ command }: ConfigEnv): UserConfigExport => {
    return {
        plugins: [
            vue(),
            viteMockServe({
                mockPath: 'mock',
                enable: true,
            }),
        ],
    }
}
```

```ts
// viteMockSer 配置
{
    mockPath?: string;
    ignore?: RegExp | ((fileName: string) => boolean);
    watchFiles?: boolean;
    enable?: boolean;
    ignoreFiles?: string[];
    configPath?: string;
    logger?:boolean;
}
```

- 生产环境
```ts
//  mockProdServer.ts
import { createProdMockServer } from 'vite-plugin-mock/client'

// 逐一导入您的mock.ts文件
// 如果使用vite.mock.config.ts，只需直接导入文件
// 可以使用 import.meta.glob功能来进行全部导入
import testModule from '../mock/test'

export function setupProdMockServer() {
  createProdMockServer([...testModule])
}
```

```ts
// vite.config.ts 配置
import { viteMockServe } from 'vite-plugin-mock'

import { UserConfigExport, ConfigEnv } from 'vite'

export default ({ command }: ConfigEnv): UserConfigExport => {
  return {
    plugins: [
      viteMockServe({
        mockPath: 'mock',
        // 根据项目配置。可以配置在.env文件
        enable: true,
      }),
    ],
  }
}
```

#### 7. 第三方云存储

- [vite-plugin-tencent-oss](https://github.com/taosiqi/vite-plugin-tencent-oss)

将项目中打包后生产文件上传到腾讯云COS，除了 html 以外。

安装：
```shell
pnpm add vite-plugin-tencent-oss -D
```

使用：
```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vitePluginTencentOss from 'vite-plugin-tencent-oss'

const options = {
  region: '<Your Region>',
  secretId: '<Your Secret ID>',
  secretKey: '<Your Secret Key>',
  bucket: '<Your Bucket>'
}

const prod = process.env.NODE_ENV === 'production'

// https://vitejs.dev/config/
export default defineConfig({
  base: prod ? 'https://foo.com/' : '/', // 打包时必须是 URL
  plugins: [vue(), vitePluginTencentOss(options)]
})
```

- [vite-plugin-ali-oss](https://github.com/xiaweiss/vite-plugin-ali-oss)

将项目中打包后生产文件上传到 Ali OSS，除了 html 以外。

安装：
```shell
pnpm add vite-plugin-ali-oss -D
```

使用：
```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vitePluginAliOss from 'vite-plugin-ali-oss'

const options = {
  region: '<Your Region>',
  accessKeyId: '<Your Access Key ID>',
  accessKeySecret: '<Your Access Key Secret>',
  bucket: '<Your Bucket>'
}

const prod = process.env.NODE_ENV === 'production'

// https://vitejs.dev/config/
export default defineConfig({
  base: prod ? 'https://foo.com/' : '/', // 打包时必须是 URL
  plugins: [vue(), vitePluginAliOss(options)]
})
```

- [vite-plugin-qiniu-oss](https://github.com/th-come/vite-plugin-qiniu-oss)

安装：
```shell
pnpm add vite-plugin-qiniu-oss -D
```

使用：
```js
import vitePluginQiniuOss from 'vite-plugin-qiniu-oss'
const uploadPath = require('./package.json').name;

export default defineConfig(() => {
  const openUpload = process.env.NODE_ENV == 'production' ? true : false

  return {
    base: openUpload ? `https://qiniu.xxx.com/${uploadPath}/`: `./`, // same with webpack public path
	plugins: [vue(), vitePluginQiniuOss(openUpload)]
  }
})

// 新建`.qiniu.config.js`配置文件，并且在 `.gitignore` 忽略此文件

const uploadPath = require('./package.json').name;
module.exports = {
  accessKey: 'qiniu access key',
  secretKey: 'qiniu secret key',
  bucket: 'demo',
  bucketDomain: 'https://domain.bkt.clouddn.com',
  uploadPath: `/${uploadPath}/`,
  batch: 10,
  zone: 'Zone_z0',
  ignore: ['**/*.html', '**/*.map']
}
```

#### 8. 依赖预构建

vite在开发环境使用了esbuild，在进入新页面时，如果有引入新的依赖浏览器才会加载新依赖。为了优化开发体验，官方支持[依赖预构建](https://cn.vitejs.dev/guide/dep-pre-bundling.html)。

所谓的预构建是为了开发时将某些依赖预先将esm转换成单个模块。这样牺牲了一些项目启动时间但是在切换页面时更流程不会等待太长时间。

> 依赖预构建仅适用于开发模式，并使用 esbuild 将依赖项转换为 ES 模块。在生产构建中，将使用 @rollup/plugin-commonjs。

```ts
// vite.config.ts
import type { UserConfig, ConfigEnv } from 'vite';
import { defineConfig, loadEnv } from 'vite';

export default defineConfig(({ mode }: ConfigEnv): UserConfig => {
    return {
        // ...其他配置项
        // 依赖优化 - 预构建
        optimizeDeps: {
            include: ['vue', 'pinia', 'vue-router', 'ant-design-vue/es', '@vueuse/core'],
        }
    }
})
```

更多配置项字段请看官方文档:[依赖优化选项](https://cn.vitejs.dev/config/dep-optimization-options.html)

### 三、更多资源

更多vite插件资源请看官方: [awesome-vite](https://github.com/vitejs/awesome-vite)
