pipeline {
  agent any

  stages {
    stage("Build") {
      when {
        not { anyOf { branch 'master' ; branch 'staging'} }
      }
      steps {
        sh "docker build --build-arg NF_IMAGE_VERSION=${env.GIT_COMMIT} ."
      }
    }

    stage("Build Untagged") {
      when {
        anyOf { branch 'master' ; branch 'staging'}
        not { buildingTag() }
      }
      steps {
        sh "docker build --build-arg NF_IMAGE_VERSION=${env.GIT_COMMIT} -t netlify/build:${env.BRANCH_NAME} -t netlify/build:${env.GIT_COMMIT} ."
        sh "docker build --build-arg NF_IMAGE_VERSION=${env.GIT_COMMIT} --squash -t netlify/build:${env.BRANCH_NAME}-squash -t netlify/build:${env.GIT_COMMIT}-squash ."
      }
    }

    stage("Build Tagged") {
      when {
        anyOf { branch 'master' ; branch 'staging'}
        buildingTag()
      }
      steps {
        sh "docker build --build-arg NF_IMAGE_VERSION=${env.GIT_COMMIT} -t netlify/build:${env.BRANCH_NAME} -t netlify/build:${env.GIT_COMMIT} -t netlify/build:${env.GIT_TAG} ."
        sh "docker build --build-arg NF_IMAGE_VERSION=${env.GIT_COMMIT} --squash -t netlify/build:${env.BRANCH_NAME}-squash -t netlify/build:${env.GIT_COMMIT}-squash -t netlify/build:${env.GIT_TAG}-squash ."
      }
    }

    stage("Push Tagged") {
      when {
        anyOf { branch 'master' ; branch 'staging'}
        buildingTag()
      }
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-ci') {
            docker.image("netlify/build:${env.BRANCH_NAME}").push()
            docker.image("netlify/build:${env.GIT_COMMIT}").push()
            docker.image("netlify/build:${env.GIT_TAG}").push()
            docker.image("netlify/build:${env.BRANCH_NAME}-squash").push()
            docker.image("netlify/build:${env.GIT_COMMIT}-squash").push()
            docker.image("netlify/build:${env.GIT_TAG}-squash").push()
          }
        }
      }
    }

    stage("Push Untagged") {
      when {
        anyOf { branch 'master' ; branch 'staging'}
        buildingTag()
      }
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-ci') {
            docker.image("netlify/build:${env.BRANCH_NAME}").push()
            docker.image("netlify/build:${env.GIT_COMMIT}").push()
            docker.image("netlify/build:${env.BRANCH_NAME}-squash").push()
            docker.image("netlify/build:${env.GIT_COMMIT}-squash").push()
          }
        }
      }
    }
  }

  post {
    failure {
      slackSend color: "danger", message: "Build failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}/console|Open>)"
    }
    success {
      slackSend color: "good", message: "Build succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}/console|Open>)"
    }
  }
}
