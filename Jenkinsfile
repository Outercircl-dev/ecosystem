pipeline {
  agent any

  options {
    disableConcurrentBuilds()
    timestamps()
  }

  environment {
    AWS_REGION        = 'eu-west-1'
    AWS_ACCOUNT_ID    = '886789338873'

    ECR_BACKEND       = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/outercircl-backend"
    ECR_FRONTEND      = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/outercircl-frontend"

    ECS_CLUSTER       = 'outercircl-staging'
    ECS_BACKEND_SVC   = 'outercircl-backend-staging-service'
    ECS_FRONTEND_SVC  = 'outercircl-frontend-staging-service'
  }

  triggers {
    pollSCM('H/5 * * * *')  // every 5 minutes – will only run when staging has changes
  }

  stages {

    stage('Checkout') {
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/staging']],
          userRemoteConfigs: [[url: 'https://github.com/Outercircl-dev/ecosystem.git']],
          extensions: [[$class: 'SubmoduleOption', recursiveSubmodules: true, trackingSubmodules: true]]
        ])

        sh 'git submodule update --init --recursive'
        sh '''
          git submodule foreach '
            git checkout main || true
            git pull origin main || true
          '
        '''
      }
    }

    stage('Set Version') {
      steps {
        script {
          env.GIT_SHA = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
          echo "Version: ${env.GIT_SHA}"
        }
      }
    }

    stage('Login to ECR') {
      steps {
        withEnv(["AWS_REGION=${env.AWS_REGION}"]) {
          sh '''
            aws ecr get-login-password --region "$AWS_REGION" \
              | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
          '''
        }
      }
    }

    stage('Build Backend Image') {
      steps {
        sh '''
          docker build -t outercircl-backend:${GIT_SHA} ./src/backend

          docker tag outercircl-backend:${GIT_SHA} ${ECR_BACKEND}:staging-${GIT_SHA}
          docker tag outercircl-backend:${GIT_SHA} ${ECR_BACKEND}:staging-latest
        '''
      }
    }

    stage('Build Frontend Image') {
      steps {
        sh '''
          docker build -t outercircl-frontend:${GIT_SHA} ./src/frontend

          docker tag outercircl-frontend:${GIT_SHA} ${ECR_FRONTEND}:staging-${GIT_SHA}
          docker tag outercircl-frontend:${GIT_SHA} ${ECR_FRONTEND}:staging-latest
        '''
      }
    }

    stage('Push Images to ECR') {
      steps {
        sh '''
          docker push ${ECR_BACKEND}:staging-${GIT_SHA}
          docker push ${ECR_BACKEND}:staging-latest

          docker push ${ECR_FRONTEND}:staging-${GIT_SHA}
          docker push ${ECR_FRONTEND}:staging-latest
        '''
      }
    }

    stage('Deploy to ECS') {
      steps {
        sh '''
          aws ecs update-service \
            --cluster ${ECS_CLUSTER} \
            --service ${ECS_BACKEND_SVC} \
            --force-new-deployment \
            --region ${AWS_REGION}

          aws ecs update-service \
            --cluster ${ECS_CLUSTER} \
            --service ${ECS_FRONTEND_SVC} \
            --force-new-deployment \
            --region ${AWS_REGION}
        '''
      }
    }
  }

  post {
    always {
      sh 'docker system prune -f || true'
    }
  }
}
