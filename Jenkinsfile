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
        echo "Running pytest with coverage (JUnit XML + coverage HTML)"
        bat '''.venv\\Scripts\\python.exe -m pytest --junitxml=reports\\junit.xml --cov=src --cov-report=xml:reports\\coverage.xml --cov-report=html:reports\\coverage_html -q'''
      }
    }

    stage('Publish & Archive') {
      steps {
        echo "Publishing JUnit, publishing coverage HTML, and archiving reports"
        // ensure reports folder exists
        bat 'if not exist reports mkdir reports'

        // publish junit test results to Jenkins
        junit 'reports\\junit.xml'

        // publish HTML coverage (requires HTML Publisher plugin)
        publishHTML([allowMissing: false,
                     alwaysLinkToLastBuild: true,
                     keepAll: true,
                     reportDir: 'reports/coverage_html',
                     reportFiles: 'index.html',
                     reportName: 'Coverage Report'])

        // archive reports for download
        archiveArtifacts artifacts: 'reports/**', fingerprint: true
      }
    }
  }

  post {
    always {
      echo "Pipeline finished: ${currentBuild.currentResult}"
    }
  }
}
