pipeline{

      agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: acavaleiro/jenkins-nodo-nodejs-bootcamp:1.0
    command:
    - sleep
    args:
    - infinity
'''
            defaultContainer 'shell'
        }
    }

  environment {
    registryCredential='acavaleiro'
    registryFrontend = 'acavaleiro/frontend-demo'
  }

  stages {
    stage('Build') {
      steps {
        sh 'npm install && npm run build'
      }
    }
  

    stage('Push Image to Docker Hub') {
      steps {
        script {
          dockerImage = docker.build registryFrontend + ":$BUILD_NUMBER"
          docker.withRegistry( '', registryCredential) {
            dockerImage.push()
          }
        }
      }
    }

    stage('Push Image latest to Docker Hub') {
      steps {
        script {
          dockerImage = docker.build registryFrontend + ":latest"
          docker.withRegistry( '', registryCredential) {
            dockerImage.push()
          }
        }
      }
    }

    stage('Deploy to K8s') {

      steps{
        script {
          if(fileExists("configuracion")){
            sh 'rm -r configuracion'
          }
        }
        sh 'git clone https://github.com/Guibrixton/kubernetes-helm-docker-config.git configuracion --branch test-implementation'
        sh 'kubectl apply -f configuracion/kubernetes-deployment/angular-14-app/manifest.yml -n default --kubeconfig=configuracion/kubernetes-config/config'
      }

    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}
