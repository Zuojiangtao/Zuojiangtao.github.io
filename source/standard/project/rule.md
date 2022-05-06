title: 开发公约
---

### 一、编码约定
- JavaScript, *.vue，Jsx代码结尾不加分号；
- 字符串默认单引号'，不用双引号"
- LESS文件内用单引号'，不用双引号"
- 比较时只能用===作比较，不用==
- 不能用var申明变量，改用let，能用const不用let
- import组件时名称为大驼峰：CompoentImportedName
- *.vue文件中name用大驼峰命名
- template模板内所有组件标签、元素标签名全部小写+连字符分隔
- 组件属性一般不用每个都换行定义，除非太多太长需要换行显示或系统已经配置了eslint
- 组件行内样式style属性不能超过3个样式属性，需要单独定义样式类
- 所有函数方法用块注释定义功能用途、参数及类型、参数说明、返回值及类型等
- Vue组件内methods中的函数之间不能空行
- 表单：不管是否采用v-decorator来申明验证规则，都需要将验证描述文件rules单独用文件封装
- 表格：所有表格列columns单独用文件封装
- 文件引用：所有文件引用尽量采用别名路径（系统已经定义了大多数别名供使用vue.config.js），除非目录间不超过2级
- *.vue模板文件内的变量尽量用scoped来限制，模块的顶层样式可以用t-xxxx 的命名来限制样式块。样式名称不要简写，尽量取名要表达的意思
- 所有模块样式颜色均来自：`~@/style/antd-custom-vars.less` 文件，通常不需要特别定义
- 使用reflect函数式语法代替in,typeof等命令式语法
- 使用语义更明确的async/await代替promise方案,是代码可读性更强
- 多级组件交互使用provide/inject，不推荐使用eventbus


### 二、http请求约定
- 目前项目都采用post提交请求， token放header，其余参数放requestBody中发送请求。
- 项目已经封装了默认的post/get方法供其它模块使用。


### 三、API定义
- API定义引用已经统一定义的函数：`import { post, get } from '@/api'`
- 在页面头部定义api路径列表，并导出

### 五、Git提交代码注释
> Git代码提交时必须添加注释，注释需要清楚的说明所提交内容的相关信息，不能过于简单。所提交内容需要添加如下11种类型的前缀加以区分注释，一目了然就知道大概做了什么操作。
> 推荐使用vscode工具git-commit-plugin进行commit message编写


* feat：新增功能
* fix：bug 修复
* docs：文档更新
* style：不影响程序逻辑的代码修改(修改空白字符，格式缩进，补全缺失的分号等，没有改变代码逻辑)
* refactor：重构代码(既没有新增功能，也没有修复 bug)
* perf：性能, 体验优化
* test：新增测试用例或是更新现有测试
* build：主要目的是修改项目构建系统(例如 glup，webpack，rollup 的配置等)的提交
* ci：主要目的是修改项目继续集成流程(例如 Travis，Jenkins，GitLab CI，Circle等)的提交
* chore：不属于以上类型的其他类型，比如构建流程, 依赖管理
* revert：回滚某个更早之前的提交



**_注：不能一次提交过多的文件，且文件包含各种修改内容。尽量多commit集中push。每天尽量多次push，不能长时间不提交代码，之后一次提交，这样会造成更多的冲突，也不便于代码跟进。_**





