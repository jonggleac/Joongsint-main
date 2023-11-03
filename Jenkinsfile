pipeline {
  agent any
  
  triggers {
    pollSCM('*/3 * * * *')
  }

  environment {
    REPOSITORY_CREDENTIAL_ID = 'gittoken'
    REPOSITORY_URL = 'https://github.com/jonggleac/Joongsint-main.git'
    TARGET_BRANCH = 'main'

    CONTAINER_NAME = 'TestOsintflask'


    AWS_CREDENTIAL_NAME = 'osintECR'

    ECR_PATH = '871065065486.dkr.ecr.us-east-2.amazonaws.com'
    IMAGE_NAME = '871065065486.dkr.ecr.us-east-2.amazonaws.com/jenkinsflask'
    REGION = 'us-east-2'
  }

  stages{
    stage('init') {
      steps {
        echo 'init stage'
        deleteDir()
      }
      post {
        success {
          echo 'Success init in pipeline'
        }
        failure {
          error 'fail init in pipeline'
        }
      }
    }

    stage('clone project') {
      steps {
        git url: "$REPOSITORY_URL",
        branch: "$TARGET_BRANCH",
        credentialsId: "$REPOSITORY_CREDENTIAL_ID"
        sh "ls -al"
      }
      post {
        success {
          echo 'success clone project'
        }
        failure {
          error 'fail clone project' // exit pipeline
        }
      }
    }

    stage('Build project') {
      steps {
        sh '''
        pip3 install -r requirements.txt

        python main.py
        '''
      }
      post {
        success {
          echo 'successfully build!'
        }
        failure {
          error 'fail to build project'
        }
      }
    }

    stage('dockerizing project') {
      steps {
        sh '''
        docker build -t $IMAGE_NAME:$BUILD_NUMBER .
        docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest
        '''
      }
      post {
        success {
          echo 'success dockerizing project'
        }
        failure {
          error 'fail dockerizing project'
          }
        }
      }
    

    stage('upload aws ECR') {
      steps {
        script {
          //cleanup cureent user docker credentials
          sh 'rm -f ~/.dockercfg ~/.docker/config.json || true'

          docker.withRegistry("https://${ECR_PATH}", "ecr:${REGION}:${AWS_CREDENTIAL_NAME}") {
            docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
            docker.image("${IMAGE_NAME}:latest").push()
          }

        }
      }
      post {
        success {
          echo 'success upload image'
        }
        failure {
          error 'fail upload image'
        }
      }
    }
  }
}