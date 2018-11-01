pipeline {
  agent {
    kubernetes {
      label 'mypod'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: docker-builder
spec:
  containers:
  - name: docker
    image: docker:18
    command:
      - cat
    tty: true
    volumeMounts:
      - name: docker
        mountPath: /var/run/docker.sock

  - name: golang
    image: golang:1.10-alpine
    command:
    - cat
    tty: true

  volumes:
    - name: docker
      hostPath:
          path: /var/run/docker.sock
"""
    }
  }
  stages {
    stage('prepare'){
         parallel{
            stage('docker env prepare'){
            steps{
                 container('docker'){
                     sh 'apk add --update make'
                 }
               }
             }
            stage('golang env pre prepare') {
            steps{
                 container('golang'){
                     sh 'apk add --update make'
                     sh 'apk add --update git'
                     sh 'apk add --no-cache bash'
                     sh 'go get golang.org/x/tools/cmd/goimports'
                 }
              }
            }
        }
    }
    stage('checkout scm & check code fmt') {
      steps {
        checkout scm
        container('golang') {
          sh 'chmod +x hack/*.sh'
          sh 'make check'
        }
      }
    }
    stage('build image') {
        steps{
        container('docker'){
          sh 'docker build . -t runzexia/kubesphere-devops-sample:latest'
        }
      }
    }
    stage('push image'){
        when { branch 'master' }
        steps{
            container('docker'){
                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                         credentialsId: 'key-23373639',
                         usernameVariable: 'DOCKER_HUB_USER',
                         passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                         sh """
                           docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
                           docker push runzexia/kubesphere-devops-sample:latest
                           """
                }
            }
        }
    }
  }

}
