title: 私库使用说明
---

### 项目打包流程

#### 1. 下载项目所需的依赖包（需连外网）

> 
- 安装下载压缩包工具，该工具是曹志剑同学根据node-tgz-downloader进行修改并上传私库
- 修改原因是node-tgz-downloader上传文件时并发操作没有限制，导致程序很容易卡死
- npm-tgz-downloader在原功能基础上改为串行执行，解决了并发卡死的问题
- api可完全参照node-tgz-downloader

```shell
# 切换至私库
npm config set registry http://10.34.252.90:8081/repository/npm-szzx/

# 安装
npm install npm-tgz-downloader -g

# 根据package.json配置文件进行压缩包的下载
npm-tgz-download package-json package.json --devDependencies
```

#### 2. 上传依赖包，查看npm包地址：http://10.34.252.90:8081/#browse/browse:npm-szzx

```shell
# 切换至私库
npm config set registry http://10.34.252.90:8081/repository/npm-szzx/

# 登录私库,账号/密码/邮箱：nexus_normal/12345678/nexus_normal@example.com
npm login

# 将npm-tgz-download下载的压缩包上传到私库
find . -type f -not -path '*/\.*' -name '*.tgz' -exec npm publish {} --registry http://10.34.252.90:8081/repository/npm-szzx/ \;
```

***完成以上两步，项目所需的依赖包就上传到了私库，之所要先上传项目依赖，是因为执行npm install时npm会解析包里的依赖，如果依赖不存在，安装时会直接报错***
***前两个步骤，我写在了npmimport.sh的脚本文件中，放在项目的根目录下，开发者需预先npm login, 之后执行脚本即可，脚本内容如下***

```shell
#!/bin/bash

# Install Download tarballs's tools
# npm install npm-tgz-downloader -g

# Downloads all of the tarballs based on one of the following
npm-tgz-download package-json package.json --devDependencies

# Set REPO_URL initial_value
REPO_URL="http://10.34.252.90:8081/repository/npm-szzx/"
# Get command line params
while getopts ":r:k:" opt; do
	case $opt in
		r) REPO_URL="$OPTARG"
		;;
	esac
done

# Set NPM Private Registry
npm config set registry $REPO_URL

# 自动登录还有问题，三套方案均会报错，请进行手动登录后再执行该脚本

# 方案1：(错误提示：verbose web login not supported, trying couch)
# printf "nexus_normal\r12345678\rnexus_normal@example.com\r" | npm login --registry $REPO_URL -ddd

# 方案2：(错误提示：verbose web login not supported, trying couch)
# npm login --registry $REPO_URL -ddd <<!
# nexus_normal
# 12345678
# nexus_normal@example.com
# !

# 方案3：(错误提示：bash: spawn: command not found)
# #!/usr/bin/expect -f
# set timeout 60
# spawn npm login --registry $REPO_URL -ddd --verbose
# match_max 100000
# expect "Username"
# send "nexus_normal\r"
# expect "Password"
# send "12345678\r"
# expect "Email"
# send "nexus_normal@example.com\r"

# 包发布
find . -type f -not -path '*/\.*' -name '*.tgz' -exec npm publish {} --registry $REPO_URL \;
```

#### 3. 下载依赖包，将依赖包上传到私库后，执行npm install从私库中直接下载依赖包

```shell
npm install
```

#### 4. 将项目进行打包

```shell
npm publish
```

***完成以上四步，我们的项目就已经成功发布成了npm包，之后在其他需要引入该包的项目中通过npm install 包名即可安装***

---

### npm包的联调
> 有时需要直接对我们自己开发npm包（geely-design-vue，geely-layout）进行联调，打完包发布后再重写引入步骤太麻烦，可使用软链接的方式进行联调

#### 1. 对包进行编译

```shell
# geely-design-vue编译命令
npm run prepublish

# geely-layout编译命令
npm run compile
```

#### 2. 生成软链接，在被依赖包的根目录下执行

```shell
# npm link命令可以将一个任意位置的npm包链接到全局执行环境，从而在任意位置使用命令行都可以直接运行该npm包
# 如在geely-design-vue包的根目录下执行
npm link
```

#### 3. 使用软链接，在需要依赖包的项目的根目录下执行，该命令作用是将全局环境中的npm包映射到项目的node_modoles目录下，包文件改变后可实时同步到node_modules

```shell
npm link 包名
# 例：npm link geely-design-vue
```

---

### 前端CI/CD 相关文件

#### 1. Jenkinsfile
> 目前五个步骤：代码拉取+分支切换 -> 依赖包安装+项目打包 -> docker容器打包 -> docker容器发布 -> 将docker部署到k8s服务上（该步骤可沿用，修改环境变量即可）

```shell
pipeline {
  agent {
    node {
      label 'nodejs'
    }
  }

    parameters {
        string(name:'TAG_NAME',defaultValue: '',description:'')
    }

	# 配置信息不同项目根据实际情况进行配置，不懂咨询曹少哲
    environment {
        DOCKER_CREDENTIAL_ID = 'dockerhub-admin'
        GITHUB_CREDENTIAL_ID = 'gitlab-csz'
        KUBECONFIG_CREDENTIAL_ID = 'kubeconfig-test'
        REGISTRY = 'harbor.geely.com'
        DOCKERHUB_NAMESPACE = 'library'
        GITHUB_ACCOUNT = 'kubesphere'
        SONAR_CREDENTIAL_ID = 'sonar-token'
        BRANCH_NAME = 'dev_0.11v'
        K8S_NAMESPACES = 'tms-project'
        APP_NAME = 'ils-tms-web'
		COMPONENT = 'ils-tms-web'
		TIER = 'frontend'
    }
    stages {
        stage ('checkout scm') {
            steps {
                checkout(scm)
            }
        }
        stage ('npm install & build') {
            steps {
                container ('nodejs') {
                    sh 'npm -v'
                    sh 'npm config set registry http://10.34.252.90:8081/repository/npm-szzx/'
                    //sh 'npm config set registry http://10.34.252.90:8081/repository/npm-public/'
                    //sh 'npm install --production'
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }
        stage ('docker build & push') {
            steps {
                container ('nodejs') {
                    sh 'docker build --no-cache -f Dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
                    withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID" ,)]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
                        sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER'
                    }
                }
            }
        }

        stage('push latest'){
           steps{
                container ('nodejs') {
                  sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
                  sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
                }
           }
        }
        stage('deploy to K8S') {
          steps {
            kubernetesDeploy(configs: 'deploy/dev-test/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
          }
        }        
    }
}

```

#### 2. Dockerfile

```shell
#FROM harbor.geely.com/library/openjdk:8-jre-alpine-basic
# 从目标仓库下载docker容器
FROM harbor.geely.com/library/nginx:lastest

# 指定nginx文件存放目录，并将项目文件和配置文件拷贝到对应目录
WORKDIR /usr/share/nginx/html
COPY deploy/default.conf /etc/nginx/conf.d/
COPY dist/ /usr/share/nginx/html

# 暴露端口号为80
EXPOSE 80
```

#### 3. nginx配置文件default.conf
> 文件放在Dockerfile里指定的目录下，该文件只需修改反向代理地址即可，其他部分可沿用

```shell
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

	# 该配置根据实际情况进行修改
    location /api {
        proxy_pass http://10.34.76.244:31001/;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```