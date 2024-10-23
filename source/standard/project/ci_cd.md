title: CI/CD
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

    environment {
        DOCKER_CREDENTIAL_ID = 'harbor-admin'
        GITHUB_CREDENTIAL_ID = 'gitlab-csz'
        KUBECONFIG_DEV_A_ID = 'kubeconfig-dev-active'
        REPLICAS = 2
        REGISTRY = 'harbor.XXXX.com'
        IMAGE_PULL_SECRET = 'harbor90'
        DOCKERHUB_NAMESPACE = 'iot'
        GITHUB_ACCOUNT = 'kubesphere'
        SONAR_CREDENTIAL_ID = 'sonar-token'
        BRANCH_NAME = 'master'
        K8S_NAMESPACES = 'workSpaceName'
        APP_NAME = 'projectName'
        COMPONENT = 'projectName'
        TIER = 'projectName'
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
                    // 设置私有node地址
                    sh 'npm config set registry http://AAAA.BBBB.CCCC:DDDD/repository/npm/'
                    sh 'npm get registry'
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
            kubernetesDeploy(configs: 'deploy/web/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_DEV_A_ID")
          }
        }
    }
}
```

#### 2. Dockerfile

```shell
#Docker地址及路径
FROM hub.cloud.XXXX.com/projectName/library/nginx:lastest

WORKDIR /usr/share/nginx/html
COPY deploy/default.conf /etc/nginx/conf.d/
COPY dist/ /usr/share/nginx/html
EXPOSE 80
```

#### 3. nginx配置文件default.conf
> 文件放在Dockerfile里指定的目录下，该文件只需修改反向代理地址即可，其他部分可沿用

```shell
# 开启gzip
gzip on;
# 启用gzip压缩的最小文件；小于设置值的文件将不会被压缩
gzip_min_length 1k;
# gzip 压缩级别 1-10
gzip_comp_level 6;
# 进行压缩的文件类型。
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
# 是否在http header中添加Vary: Accept-Encoding，建议开启
gzip_vary on;

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

    location /api/v1/ {
        proxy_pass http://www.baidu.com/api/v1/;
        client_max_body_size 8M;
        client_body_buffer_size 8M;
        fastcgi_intercept_errors on;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
