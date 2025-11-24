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
        bat 'python -m venv .venv'
        bat '.venv\\Scripts\\python.exe -m pip install --upgrade pip'
        bat '.venv\\Scripts\\python.exe -m pip install -r requirements.txt'
      }
    }

    stage('Run Tests with Coverage') {
      steps {
        echo "Running pytest with coverage"
        // produce junit xml, coverage XML and HTML report
        bat '''.venv\\Scripts\\python.exe -m pytest --junitxml=reports\\junit.xml --cov=src --cov-report=xml:reports\\coverage.xml --cov-report=html:reports\\coverage_html -q'''
      }
    }

    stage('Publish & Archive') {
      steps {
        echo "Publishing JUnit, archiving artifacts, publishing coverage HTML"
        // Ensure reports folder exists before Jenkins tries to process
        bat 'if not exist reports mkdir reports'

        // Publish junit results (Jenkins built-in)
        junit 'reports\\junit.xml'

        // Archive coverage files and HTML so you can download them
        archiveArtifacts artifacts: 'reports/**', fingerprint: true

        // Publish HTML coverage using HTML Publisher plugin (configured below)
        // HTML Publisher step will be run by plugin in post build — below is instructions to configure it in Jenkins UI
      }
    }
  }

  post {
    always {
      echo "Pipeline finished: ${currentBuild.currentResult}"
    }
  }
}
