pipeline {
  agent any

  environment {
    IMAGE_NAME = 'mehaknarang0609/simple-node-app'
    DOCKER_CREDENTIALS = credentials('dockerhub-creds')
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/mehak-World/NodeDemo.git'
      }
    }

    stage('Install & Test') {
      steps {
        sh 'npm install'
        // Add tests here if you have any
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t $IMAGE_NAME ."
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withDockerRegistry([credentialsId: 'dockerhub-creds', url: '']) {
          sh "docker push $IMAGE_NAME"
        }
      }
    }

    stage('Provision EC2') {
      steps {
        dir('terraform') {
          sh '''
            terraform init
            terraform apply -auto-approve
          '''
        }
      }
    }

stage('Get EC2 Public IP') {
      steps {
        script {
          dir('terraform') {
            // Capture the public IP in a Jenkins variable
            EC2_IP = sh(script: "terraform output -raw public_ip", returnStdout: true).trim()
            echo "EC2 Public IP: ${EC2_IP}"
          }
        }
      }
    }

    stage('Deploy to EC2') {
      steps {
        sshagent(['ec2-key']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ec2-user@<EC2_IP> "
              docker pull $IMAGE_NAME &&
              docker run -d -p 80:3000 $IMAGE_NAME
            "
          '''
        }
      }
    }
  }
}
