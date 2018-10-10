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
        }
    }
    stage('scm'){
        steps {
            git 'https://github.com/runzexia/kubesphere-devops-sample'
        }
    }
    stage('check') {
      steps {
        container('docker') {
          sh 'make fmt-check'
        }
      }
    }
    stage('build image') {
        steps{
        container('docker') {
          sh 'make build'
        }
      }
    }
  }

}
