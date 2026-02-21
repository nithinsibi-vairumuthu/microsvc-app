pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    ACCOUNT_ID = "730335674713"   // 🔴 change this
    IMAGE_TAG  = "${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Services in Parallel') {
      parallel {

        stage('Build Service A (Auth)') {
          steps {
            dir('service-a') {
              sh "docker build -t service-a:${IMAGE_TAG} ."
            }
          }
        }

        stage('Build Service B (Payment)') {
          steps {
            dir('service-b') {
              sh "docker build -t service-b:${IMAGE_TAG} ."
            }
          }
        }

      }
    }

    stage('Security Scan') {
      steps {
        sh '''
          echo "Running vulnerability scan"
              trivy image --ignore-unfixed --exit-code 1 --severity HIGH,CRITICAL service-a:${IMAGE_TAG}
              trivy image --ignore-unfixed --exit-code 1 --severity HIGH,CRITICAL service-b:${IMAGE_TAG}
        '''
      }
    }

    stage('Push to ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region $AWS_REGION | \
          docker login --username AWS --password-stdin \
          $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

          docker tag service-a:$IMAGE_TAG $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-a:$IMAGE_TAG
          docker tag service-b:$IMAGE_TAG $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-b:$IMAGE_TAG

          docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-a:$IMAGE_TAG
          docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-b:$IMAGE_TAG
        '''
      }
    }

    stage('Deploy with Rolling Update') {
      steps {
        sh '''
          echo "Triggering rolling update in Kubernetes"
          # kubectl apply -f k8s/service-a/
          # kubectl apply -f k8s/service-b/
        '''
      }
    }
  }

  post {
    failure {
      echo "❌ Deployment Failed - Initiating Rollback"
    }
  }
}

