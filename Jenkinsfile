pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  tools {
    go 'go'
  }
  stages {
    stage('Clean Workspace') {
        steps {
            cleanWs() // Clean the workspace
        }
    }
    stage('Checkout') {
        steps {
            
            withCredentials([usernamePassword(credentialsId: 'xeni-admin-github-creds', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_PASS')]) {
                // Clone the repository using the stored credentials
                sh "git clone https://${GITHUB_PASS}@github.com/anas-xeni/simple-web-server ."
            }

        }
    }

    stage('go build'){
      steps{
        script {
            sh 'go version'
            // sh 'go get -d github.com/gorilla/mux'
            // sh 'go get -d github.com/prometheus/client_golang/prometheus'
            // sh 'go get -d github.com/prometheus/client_golang/prometheus/promhttp'
            sh 'go mod init simple-web-server'
            sh 'go mod tidy'
            sh 'export GOOS=linux && export GOARCH=arm64 && export CGO_ENABLED=0 && go build -a -installsuffix cgo --ldflags "-s -w" -o server'
        }
      }
    }
  

    stage('docker build'){
      steps{
        script {
          docker.build('densikatshine/simple-web-server')
        }
      }
    }

    // stage('docker push'){
    //   steps{
    //     script {
    //       docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
    //         docker.image('densikatshine/simple-web-server').push('badconfig')
    //       }
    //     }
    //   }
    // }

    stage('Login to ECR') {
      steps {
        script {
          // Login to ECR
          sh "aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 887376482243.dkr.ecr.us-west-2.amazonaws.com"
        }
      }
    }

    stage('Tag and Push Docker Image') {
      steps {
        script {
          // Tag the Docker image
          sh "docker tag densikatshine/simple-web-server 887376482243.dkr.ecr.us-west-2.amazonaws.com/hello-world-for-spinnaker:${env.BUILD_ID}-${env.GIT_COMMIT}"
          // Push the Docker image
          sh "docker push 887376482243.dkr.ecr.us-west-2.amazonaws.com/hello-world-for-spinnaker:${env.BUILD_ID}-${env.GIT_COMMIT}"
        }
      }
    }
  }
}
