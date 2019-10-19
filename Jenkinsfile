pipeline {
  agent any
  stages {
    stage('docker build') {
      steps {
        sh 'docker build -t ${DOCKER_REGISTRY_HOST}/${NAME}:${TAG} .'
      }
    }
    stage('docker push') {
      steps {
        sh 'docker login -u ${DOCKER_REGISTRY_CREDS_USR} -p ${DOCKER_REGISTRY_CREDS_PSW} ${DOCKER_REGISTRY_HOST}'
        sh 'docker push ${DOCKER_REGISTRY_HOST}/${NAME}:${TAG}'
      }
    }
    stage('docker clean') {
      steps {
        sh 'echo "clean none image"'
        sh 'docker rmi $(docker images| grep "<none>" | awk "{print $3}")'
      }
    }
    stage('deploy') {
      agent {
        docker {
          image 'lwolf/helm-kubectl-docker'
        }
      }
      steps {
        sh "mkdir -p ~/.kube"
        sh "echo ${K8S_CONFIG} | base64 -d > ~/.kube/config"
        sh "./deploy.sh"
      }
    }
  }
  environment {
    DOCKER_REGISTRY_CREDS = credentials('docker-hub-registry-auth-id')
    K8S_CONFIG = credentials('kubernetes-config-secret-id')
    DOCKER_REGISTRY_HOST = 'docker.io/tomoncleshare'
    NAME = 'jenkins-python'
    TAG = 'v1.0'
  }
}