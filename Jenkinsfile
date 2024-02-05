pipeline {
  agent any
 
  stages {
    stage('Install sam-cli') {
      steps {
        sh 'python3 -m venv venv && venv/bin/pip install aws-sam-cli'
        stash includes: '**/venv/**/*', name: 'venv'
      }
    }
    stage('Build') {
      steps {
        unstash 'venv'
        sh 'venv/bin/sam build'
        stash includes: '**/.aws-sam/**/*', name: 'aws-sam'
      }
    }
    stage('Deploy') {
      environment {
        STACK_NAME = 'sam-app-prod-stage'
        S3_BUCKET = 'tools-selection-app'
      }
      steps {
        script {
          def stackStatus = sh(script: 'aws cloudformation describe-stacks --stack-name $STACK_NAME --query "Stacks[0].StackStatus" --output text', returnStatus: true).trim()

          if (stackStatus == "ROLLBACK_COMPLETE") {
            echo "Stack is in ROLLBACK_COMPLETE state. Deleting the stack..."
            sh "aws cloudformation delete-stack --stack-name $STACK_NAME"
            sh "aws cloudformation wait stack-delete-complete --stack-name $STACK_NAME"
          }

          echo "Deploying the stack..."
          withAWS(credentials: 'sam-jenkins-demo-credentials', region: 'us-east-1') {
            unstash 'venv'
            unstash 'aws-sam'
            sh 'venv/bin/sam deploy --stack-name $STACK_NAME -t template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'
          }
        }
      }
    }
  }
}
