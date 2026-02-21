pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    ACCOUNT_ID = "730335674713"   // 🔴 change this
    VERSION    = "v1.0.${BUILD_NUMBER}"   // simple auto-version for now
    GIT_SHA    = "${env.GIT_COMMIT}".take(7)
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
              sh "docker build --no-cache -t service-a:build ."
            }
          }
        }

        stage('Build Service B (Payment)') {
          steps {
            dir('service-b') {
              sh "docker build --no-cache -t service-b:build ."
            }
          }
        }

      }
    }

    stage('Security Scan') {
      steps {
        sh '''
          echo "Running vulnerability scan"
              trivy image --ignore-unfixed --exit-code 1 --severity HIGH,CRITICAL service-a:build
              trivy image --ignore-unfixed --exit-code 1 --severity HIGH,CRITICAL service-b:build
        '''
      }
    }

    stage('Push to ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region $AWS_REGION | \
          docker login --username AWS --password-stdin \
          $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

          docker tag service-a:build $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-a:$VERSION
          docker tag service-a:build $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-a:$GIT_SHA

          docker tag service-b:build $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-b:$VERSION
          docker tag service-b:build $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-b:$GIT_SHA

          docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-a:$VERSION
          docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-a:$GIT_SHA
          
          docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-b:$VERSION
          docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/service-b:$GIT_SHA
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

