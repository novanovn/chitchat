pipeline {

  environment {
    dockerregistry = 'https://registry.hub.docker.com'
    dockerhuburl = 'novanovn/chitchat'
    githuburl = 'novanovn/chitchat'
    dockerhubcrd = '94992ee7-67a1-4a48-8086-d9a1d2d1ddb3'
    dockerImage = ''
  }

  agent any

  tools {nodejs "nodejs1017"}

  stages {

    stage('Clone git repo') {
      steps {
         git 'https://github.com/' + githuburl
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'npm install'
      }
    }     
    stage('Test') {
      steps {
         sh 'npm test'
        }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build(dockerhuburl + ":$BUILD_NUMBER")
        }
      }
    }

    stage('Test image') {
      steps {
        sh 'docker run -i ' + dockerhuburl + ':$BUILD_NUMBER npm test'
      }
    }

    stage('Deploy image') {
      steps{
        script {
          docker.withRegistry(dockerregistry, dockerhubcrd ) {
            dockerImage.push("${env.BUILD_NUMBER}")
            dockerImage.push("latest")
          }
        }
      }
    }

    stage('Remove image') {
      steps{
        sh "docker rmi $dockerhuburl:$BUILD_NUMBER"
      }
    }

    stage('Deploy k8s') {
      steps {
        kubernetesDeploy(
          kubeconfigId: '534dde19-d90a-4049-83b3-c1c391365c12',
          configs: 'k8s.yaml',
          enableConfigSubstitution: true
        )
      }
    }
  }
}
