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
  libraries {
    lib('DaytraderLib')
  }
  stages {
    stage('build maven') {
        steps {
            mavenBuild('daytrader-gatewayapp')
        }
    }
        stage('Build with Kaniko') {
      environment {
        PATH = "/busybox:/kaniko:$PATH"
      }
      steps {
        kanikoBuild('kaniko',
                    'daytrader-gatewayapp',
                    'daytrader-gateway',
                    'baserepodev.devrepo.malibu-pctn.com/104017-malibu-artifacts',
                    'daytrader-example-gatewayapp',
                    'latest',
                    '4.0.0', 
                    2443)
      }
    }
  }
}
