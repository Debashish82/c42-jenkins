pipeline {
  agent {
    label 'worker-c42'
  }

  stages {
    stage('Git Checkout') {
      steps {
        checkout([$class: 'GitSCM',
                  branches: [[name: '*/main']],
                  userRemoteConfigs: [[url: 'https://github.com/Debashish82/c42-jenkins.git']]])
      }
    }

     stage('Stopping the container') {
      steps {
        script {
          sh 'docker kill $(docker ps -q)'
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          sh '''
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 764242751754.dkr.ecr.us-east-1.amazonaws.com
          docker build -t upgrad-deb82:${BUILD_NUMBER} .
          docker tag upgrad-deb82:latest 764242751754.dkr.ecr.us-east-1.amazonaws.com/upgrad-deb82:${BUILD_NUMBER}
          docker push 764242751754.dkr.ecr.us-east-1.amazonaws.com/upgrad-deb82:${BUILD_NUMBER}
          '''
        }
      }
    }

    stage('Cleanup the docker image') {
      steps {
        script {
          sh 'docker rmi 764242751754.dkr.ecr.us-east-1.amazonaws.com/upgrad-deb82:${BUILD_NUMBER}
          sh 'docker rmi upgrad-deb82:${BUILD_NUMBER}'
        }
      }
    }

    stage('Deploy the application') {
      steps {
        script {
          sh '''
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 764242751754.dkr.ecr.us-east-1.amazonaws.com
          docker pull 764242751754.dkr.ecr.us-east-1.amazonaws.com/upgrad-deb82:${BUILD_NUMBER}
          docker run -d -p 8080:8081 764242751754.dkr.ecr.us-east-1.amazonaws.com/upgrad-deb82:${BUILD_NUMBER}
          '''
        }
      }
    }
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
    timeout(time: 1, unit: 'HOURS')
  }
}
