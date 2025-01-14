title: npm镜像源配置
---

### 一、国内npm镜像源
1. **淘宝NPM镜像**
   https://registry.npmmirror.com/
2. **阿里云NPM镜像**
   https://npm.aliyun.com/
3. **腾讯云NPM镜像**
   https://mirrors.cloud.tencent.com/npm/
4. **华为云NPM镜像**
   https://repo.huaweicloud.com/repository/npm/ （需要自己开通）
5. **网易NPM镜像**
   https://mirrors.163.com/npm/
6. **中国科学技术大学开源NPM镜像**
   https://mirrors.ustc.edu.cn/npm/
7. **清华大学开源NPM镜像**
   https://mirrors.tuna.tsinghua.edu.cn/npm/ （需要镜像申请）
8. **CNPM镜像**
   https://r.cnpmjs.org/

### 二、nrm介绍
nrm是一个npm仓库源切换工具，用于不同镜像仓库的地址切换管理。

> nrm仓库：https://github.com/Pana/nrm

#### 全局安装

```shell
# npm
npm install nrm -g

# yarn
yarn global add nrm

# pnpm
pnpm add nrm -g
```

#### 查看镜像仓库列表
```shell
$ nrm ls

* npm ---------- https://registry.npmjs.org/
  yarn --------- https://registry.yarnpkg.com/
  tencent ------ https://mirrors.tencent.com/npm/
  cnpm --------- https://r.cnpmjs.org/
  taobao ------- https://registry.npmmirror.com/
  npmMirror ---- https://skimdb.npmjs.com/registry/
  huawei ------- https://repo.huaweicloud.com/repository/npm/

```

#### 使用某个镜像仓库
```shell
nrm use <registry>
```

#### 增加新的仓库
```shell
nrm add <registry> <url>
```

比如：将上述清华大学的镜像加入到列表可以执行 `nrm add tsinghua https://mirrors.tuna.tsinghua.edu.cn/npm/` 即可。

#### 删除镜像仓库
```shell
nrm del <registry>
```

其他的命令可看 https://github.com/Pana/nrm?tab=readme-ov-file

### 三、项目级npm镜像配置
#### 设置全局源
通过上述 `nrm use` 命令或直接更改本地npm仓库地址 `npm config set registry <url>`

#### 设置项目源
在项目根目录下创建 `.npmrc` 文件，通过配置仓库地址，可将某些依赖指向自己的私有仓库。

```shell
registry=https://registry.npmmirror.com
```

#### 设置空间源
```shell
@my-group:registry=https://registry.npmmirror.com
```
这个配置可对某个命名空间下的所有包生效。例如：@vitejs/xxx。
