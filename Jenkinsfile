pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        echo "Checking out from GitHub"
        checkout scm
      }
    }
    stage('Build') {
      steps {
        echo "Simple build step (Window)"
        bat 'echo Hello from Jenkinsfile && dir'
      }
    }
  }
  post {
    always { echo "Pipeline finished with status: ${currentBuild.currentResult}" }
  }
}
