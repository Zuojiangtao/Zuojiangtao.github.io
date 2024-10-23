title: 文档指南
---

### 官方文档

[YApi官网](https://hellosean1025.github.io/yapi/index.html)

### YApi特性

- 权限管理
- 可视化接口管理
- Mock Server
- 自动化测试
- 数据导入
- 插件机制
- 免费开源，内网部署

### 内网部署：

#### 环境要求
* nodejs（7.6+)
* mongodb（2.6+）
* git

#### 安装
1. 方式一. 可视化部署[推荐]

*_执行 yapi server 启动可视化部署程序，输入相应的配置和点击开始部署，就能完成整个网站的部署。部署完成之后，可按照提示信息，执行 node/{网站路径/server/app.js} 启动服务器。在浏览器打开指定url, 点击登录输入您刚才设置的管理员邮箱，默认密码(ymfe.org) 登录系统（默认密码可在个人中心修改）。_*

```
npm install -g yapi-cli --registry https://registry.npm.taobao.org
yapi server
```

> 注意在部署时，命令行窗口不要关闭，并且要运行mongo数据库

2. 方式二. 命令行部署

*_如果 github 压缩文件无法下载，或需要部署到一些特殊的服务器，可尝试此方法_*

```markdown
mkdir yapi
cd yapi
git clone https://github.com/YMFE/yapi.git vendors //或者下载 zip 包解压到 vendors 目录（clone 整个仓库大概 140+ M，可以通过 `git clone --depth=1 https://github.com/YMFE/yapi.git vendors` 命令减少，大概 10+ M）
cp vendors/config_example.json ./config.json //复制完成后请修改相关配置
cd vendors
npm install --production --registry https://registry.npm.taobao.org
npm run install-server //安装程序会初始化数据库索引和管理员账号，管理员账号名可在 config.json 配置
node server/app.js //启动服务器后，请访问 127.0.0.1:{config.json配置的端口}，初次运行会有个编译的过程，请耐心等候
```

3. 方式三. docker部署（非官方）
* [使用 alpine 版 docker 镜像快速部署 yapi](https://www.jianshu.com/p/a97d2efb23c5)
* [docker-yapi: 基于官方yapi-cli的docker-compose方案](https://github.com/Ryan-Miao/docker-yapi)
* [docker-compose一键部署yapi](https://github.com/jinfeijie/yapi)
* [docker-YApi: 更易用的 YApi 镜像](https://github.com/fjc0k/docker-YApi)
* [使用DockerCompose构建部署Yapi](https://github.com/MyHerux/daily-code/blob/master/Program/%E5%B7%A5%E5%85%B7%E7%AF%87/Yapi/%E4%BD%BF%E7%94%A8DockerCompose%E6%9E%84%E5%BB%BA%E9%83%A8%E7%BD%B2Yapi.md)
* [yapi-docker: dockerized yapi deployment all in one](https://github.com/williamlsh/yapi-docker)


#### 更多关于YApi的相关介绍：
- [Yapi：Windows本地部署](https://blog.csdn.net/J_____Q/article/details/122009936)

### 使用教程
[YApi-使用教程](https://hellosean1025.github.io/yapi/documents/index.html)

### 接口测试插件
[yapi chrome插件cross-request](https://juejin.cn/post/6844904057711099912)
