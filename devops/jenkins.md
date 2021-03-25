# Jenkins

## 流水线(pipeline)

Jenkins 流水线 (或简单的带有大写"P"的"Pipeline") 是一套插件，它支持实现和集成 continuous delivery pipelines 到Jenkins。

Jenkins 流水线的定义可以到一个文件中 ( Jenkinsfile)保存到源代码的版本控制库中

## 示例
```jenkins
pipeline {
  agent none
  stages {
    stage('Prepare') {
      steps {
        echo 'Prepare...'
      }
    }
    stage('Build') {
      agent {
        docker {
          image 'golang'
        }
      }
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '$COMMIT']], extensions: [], userRemoteConfigs: [[credentialsId: 'ca9968d1-3d82-453c-8380-27a1bb300a2d', url: 'git@github.com:iscod/IsCod.github.io.git']]])
        sh 'ls -lh'
        sh 'go mod download'
        sh 'go build -a -v -o tao-gin'
      }
    }
    stage('Test') {
        agent any
        steps {
            sh 'ls -lh'
        }
    }
    stage('Deploy') {
        agent any
        steps {
            sh 'if ([ "$ENV" != "" ]); then mv conf/gin.$ENV.conf conf/gin.conf; fi'
            sh 'docker build  -t $JOB_BASE_NAME:$BUILD_NUMBER  .'
        }
    }
  }
}
``` 

* 参考
    * [Jenkins](https://www.jenkins.io/zh/doc/book/pipeline/)