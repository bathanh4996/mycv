pipeline {
  agent any
  environment {
    FRONTEND_GIT = 'https://github.com/bathanh4996/mycv.git'
    FRONTEND_BRANCH = 'master'
    FRONTEND_IMAGE = 'bathanh4996/cv-frontend'
    FRONTEND_SERVER = '35.187.228.188'
  }
  stages {
    stage('Build Code') {
      steps {
        git(url: FRONTEND_GIT, branch: FRONTEND_BRANCH)
        stash(name: 'frontend', includes: '*')
      }
    }
    stage('Build Image') {
      steps {
        unstash 'frontend'
        script {
          docker.withRegistry('', 'docker-hub') {
            def image = docker.build(FRONTEND_IMAGE)
            image.push(BUILD_ID)
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        script {
          withCredentials([sshUserPrivateKey(
            credentialsId: 'ssh',
            keyFileVariable: 'identityFile',
            passphraseVariable: '',
            usernameVariable: 'user'
          )]) {
            def remote = [:]
            remote.name = 'server'
            remote.host = FRONTEND_SERVER
            remote.user = user
            remote.identityFile = identityFile
            remote.allowAnyHosts = true

            sshCommand remote: remote, command: "cd $FRONTEND_SERVER_DIR && export FRONTEND_IMAGE=$FRONTEND_IMAGE:$BUILD_ID && docker-compose up -d"
          }
        }
      }
    }
  }
}