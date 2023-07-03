pipeline {
  agent {
    label 'worker'
  }
   
  stages {
    stage('Git Checkout') {
      steps {
        checkout([$class: 'GitSCM',
                  branches: [[name: '*/main']],
                  userRemoteConfigs: [[url: 'https://github.com/vbprashob/jenkins-c42-prashob']]])
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
		  aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 570161380898.dkr.ecr.us-east-1.amazonaws.com
		  docker build -t c42prashobecr:${BUILD_NUMBER} .
		  docker tag c42prashobecr:${BUILD_NUMBER} 570161380898.dkr.ecr.us-east-1.amazonaws.com/c42prashobecr:${BUILD_NUMBER}
          docker push 570161380898.dkr.ecr.us-east-1.amazonaws.com/c42prashobecr:${BUILD_NUMBER}
          '''
        }
      }
    }

    stage('Cleanup the docker image') {
      steps {
        script {
          sh 'docker rmi 570161380898.dkr.ecr.us-east-1.amazonaws.com/c42prashobecr:${BUILD_NUMBER}'
          sh 'docker rmi c42prashobecr:${BUILD_NUMBER}'
        }
      }
    }

    stage('Deploy the application') {
      steps {
        script {
          sh '''
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 570161380898.dkr.ecr.us-east-1.amazonaws.com
          docker pull 570161380898.dkr.ecr.us-east-1.amazonaws.com/c42prashobecr:${BUILD_NUMBER}
          docker run -d -p 8080:8081 570161380898.dkr.ecr.us-east-1.amazonaws.com/c42prashobecr:${BUILD_NUMBER}
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
