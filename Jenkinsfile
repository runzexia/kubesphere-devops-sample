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
    image: golang:1.10.3
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
    stage('pre'){
        steps{
            container('docker'){
                sh 'apk add --update make'
            }
            container('golang'){
                sh 'go get golang.org/x/tools/cmd/goimports'
            }
        }
    }
    stage('scm'){
        steps {
            git 'https://github.com/runzexia/kubesphere-devops-sample'
        }
    }
    stage('check') {
      steps {
        container('golang') {
          sh 'goimports -l -w -e -local=kubesphere -srcdir=/go/src/kubesphere.io/kubesphere ./cmd ./pkg'
        }
      }
    }
    stage('build image') {
        steps{
        container('golang') {
          sh 'make install'
        }
        container('docker'){
          sh 'make build'
        }
      }
    }
  }

}
