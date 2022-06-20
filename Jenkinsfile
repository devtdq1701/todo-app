@Library('notify-job-result')_
pipeline {

  agent none //khong khai bao
  environment {
    DOCKER_IMAGE = "quangtran1701/todo-app"
  }

  stages {
    stage( 'Clone' ) {
      agent { node {label 'build'}}
      steps {
        git branch: 'main',
            credentialsId: 'dangquang-github',
            url: 'https://gitlab.com/devtdq1701/todo-app.git'
        }
      }
    // stage("Test") {
    //   agent {
    //       docker {
    //         image 'python:3.8-slim-buster'
    //         args '-u 0:0 -v /tmp:/root/.cache'
    //       }
    //     }
    //   steps {
    //     sh "pip install poetry"
    //     sh "poetry install"
    //     sh "poetry run pytest"
    //     }
    // }

    stage("build") {
      agent { node {label 'build'}}
      environment {
        DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
      }
      steps {
        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
        sh "docker image ls | grep ${DOCKER_IMAGE}"
        withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
            sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
            sh "docker push ${DOCKER_IMAGE}:latest"
        }

        //clean to save disk
        sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
        sh "docker image rm ${DOCKER_IMAGE}:latest"
      }
    }

    stage("deploy") {
      agent { node {label 'dev'}}
      steps {
        sh "docker rm -vf todo-app"
        sh "docker pull ${DOCKER_IMAGE}:latest"
        sh "docker run -d -p 8080:8080 --name todo-app ${DOCKER_IMAGE}:latest"
      }
    }
  }
  post {
        failure {
          sendTelegram("Log: ${BUILD_URL}console\nJob: ${JOB_NAME}\nBuild # ${BUILD_NUMBER}\nStatus: failure")
        }
        success {
          sendTelegram("Log: ${BUILD_URL}console\nJob: ${JOB_NAME}\nBuild # ${BUILD_NUMBER}\nStatus: success")
        }
  }
}
