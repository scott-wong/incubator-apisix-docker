pipeline {
    agent {
        node {
            label 'base'
        }
    }

    environment {
        // app label
        PROJECT = 'apisix'
        NAMESPACE = 'cargo-dev'
        // 服务唯一标识
        APP_NAME = 'apisix'
        VERSION = '1.2'

        // 镜像密钥，官方dockerhub-cr，阿里云aliyun-cr 海航云 hna-cr
        DOCKER_SECRETS = 'hna-cr'

        // 以下是devops公共配置
        KUBECONFIG_CREDENTIAL_ID = 'cargo-kubeconfig'
        // 官方dockerhub-id，阿里云镜像用aliyun-id，海航云镜像用hna-id
        DOCKER_CREDENTIAL_ID = 'hna-id'
        // 阿里云registry.cn-hangzhou.aliyuncs.com，海航云bjdhub.haihangyun.com
        REGISTRY = 'bjdhub.haihangyun.com'
        // 阿里云私有:wangscott，公共&dockerhub:scottwong，海航云:wangyu5
        DOCKERHUB_NAMESPACE = 'wangyu5'
    }

    stages {
        stage ('checkout scm') {
            steps {
                checkout(scm)
            }
        }

        stage ('build & push') {
            steps {
                container ('base') {
                    sh 'docker build -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$VERSION --build-arg APISIX_VERSION=$VERSION -f centos/Dockerfile-cn centos'
                    withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID" ,)]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" $REGISTRY --password-stdin'
                        sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$VERSION'
                        sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$VERSION $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
                        sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
                    }
                }
            }
        }

        stage('deploy to dev') {
            steps {
                kubernetesDeploy(configs: 'deploy/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
            }
        }
    }
}