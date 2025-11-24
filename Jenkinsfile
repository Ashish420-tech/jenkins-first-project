pipeline {
  agent any

  stages {

    stage('Checkout') {
      steps {
        checkout scm
        echo "Checked out successfully"
      }
    }

    stage('Setup Python') {
      steps {
        echo "Create venv and install deps"
        // create venv
        bat 'python -m venv .venv'

        // upgrade pip CORRECTLY
        bat '.venv\\Scripts\\python.exe -m pip install --upgrade pip'

        // install requirements
        bat '.venv\\Scripts\\python.exe -m pip install -r requirements.txt'
      }
    }

    stage('Run Tests') {
      steps {
        echo "Running pytest"
        bat '.venv\\Scripts\\python.exe -m pytest --junitxml=reports\\junit.xml'
      }
    }

    stage('Archive & Publish') {
      steps {
        junit 'reports\\junit.xml'
        archiveArtifacts artifacts: 'dist/**', fingerprint: true
      }
    }
  }

  post {
    always {
      echo "Pipeline finished: ${currentBuild.currentResult}"
    }
  }
}
