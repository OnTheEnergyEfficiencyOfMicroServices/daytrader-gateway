pipeline {
  agent {
    kubernetes {
      //cloud 'kubernetes'
      label 'mypod'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.6-jdk-8
    command: ['cat']
    tty: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /kaniko/.docker
  volumes:
  - name: jenkins-docker-cfg
    projected:
      sources:
      - secret:
          name: devrepo-secret
          items:
            - key: .dockerconfigjson
              path: config.json
"""
    }
  }
  stages {
    stage('build maven') {
        steps {
            container('maven') {
                dir ('daytrader-gatewayapp') {
                    sh 'mvn package -B -e -Dmaven.test.skip=true'
                    //sh 'pwd'
                    //sh 'ls -R | grep target'
                }
                //sh 'pwd'
                //sh 'ls -la'
                //sh 'ls -la daytrader-gatewayapp/daytrader-gateway/target'
            }
        }
    }
        stage('Build with Kaniko') {
      environment {
        PATH = "/busybox:/kaniko:$PATH"
      }
      steps {
        container(name: 'kaniko', shell: '/busybox/sh') {
            sh '''#!/busybox/sh
            /kaniko/executor -v debug -f `pwd`/daytrader-gatewayapp/daytrader-gateway/Dockerfile -c `pwd`/daytrader-gatewayapp/daytrader-gateway --insecure --skip-tls-verify --destination=baserepodev.devrepo.malibu-pctn.com/104017-malibu-artifacts/daytrader-example-gatewayapp:latest \
            --build-arg WAR_ARTIFACTID=daytrader-gateway \
            --build-arg APP_VERSION=4.0.0 \
            --build-arg APP_ARTIFACTID=daytrader-gatewayapp \
            --build-arg EXPOSE_PORT=2443
            '''
            
        }
      }
    }
  }
}
