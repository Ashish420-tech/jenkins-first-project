// Jenkinsfile: Full, Windows-friendly declarative pipeline for Python CI
// - Creates venv, installs deps
// - Runs pytest with coverage (XML + HTML)
// - Publishes JUnit test results and HTML coverage (requires HTML Publisher plugin)
// - Builds Docker image and optionally pushes to Docker Hub using stored credentials
// Uploaded original file for reference: /mnt/data/#13.txt

pipeline {
  agent any

  parameters {
    booleanParam(name: 'RUN_DOCKER', defaultValue: false, description: 'Build Docker image?')
    booleanParam(name: 'PUSH_DOCKER', defaultValue: false, description: 'Push Docker image to registry (requires credentials)')
    string(name: 'DOCKER_IMAGE', defaultValue: 'ashish420tech/python-ci-image', description: 'Docker image name (repo/name)')
  }

  environment {
    REPORT_DIR = 'reports'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        echo "Checked out: ${env.GIT_COMMIT}"
      }
    }

    stage('Setup Python') {
      steps {
        echo 'Create virtual environment and install dependencies'
        bat 'python -m venv .venv'
        bat '.venv\\Scripts\\python.exe -m pip install --upgrade pip'
        bat '.venv\\Scripts\\python.exe -m pip install -r requirements.txt'
      }
    }

    stage('Run Tests with Coverage') {
      steps {
        echo 'Running pytest with coverage (JUnit XML + coverage HTML)'
      stage('Run Tests with Coverage') {
  steps {
    echo 'Running pytest with coverage (JUnit XML + coverage HTML)'
    bat """.venv\\Scripts\\python.exe -m pytest --junitxml=reports\\junit.xml --cov=src --cov-report=xml:reports\\coverage.xml --cov-report=html:reports\\coverage_html -q"""
  }
}
  bat '''.venv\\Scripts\\python.exe -m pytest --junitxml=reports\\junit.xml --cov=src --cov-report=xml:${REPORT_DIR}\\coverage.xml --cov-report=html:${REPORT_DIR}\\coverage_html -q'''
      }
    }

    stage('Publish Results') {
      steps {
        echo 'Publishing test results and archiving reports'
        bat "if not exist ${REPORT_DIR} mkdir ${REPORT_DIR}"
        junit "${REPORT_DIR}\\junit.xml"
        // Publish HTML coverage report (requires HTML Publisher plugin)
        publishHTML([allowMissing: false,
                     alwaysLinkToLastBuild: true,
                     keepAll: true,
                     reportDir: "${REPORT_DIR}/coverage_html",
                     reportFiles: 'index.html',
                     reportName: 'Coverage Report'])
        archiveArtifacts artifacts: '${REPORT_DIR}/**', fingerprint: true
      }
    }

    stage('Docker Build & Push') {
      when {
        expression { return params.RUN_DOCKER == true }
      }
      steps {
        script {
          def shortCommit = env.GIT_COMMIT ? env.GIT_COMMIT.substring(0,7) : 'local'
          def imageTag = "${params.DOCKER_IMAGE}:${shortCommit}"

          echo "Building Docker image: ${imageTag}"
          // Build image
          bat "docker build -t ${imageTag} ."

          // Optionally push using stored Jenkins credentials (username/password)
          if (params.PUSH_DOCKER) {
            withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
              echo 'Logging into Docker registry'
              bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
              echo "Pushing image: ${imageTag}"
              bat "docker push ${imageTag}"
            }
          }
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline succeeded: ${env.BUILD_URL}"
    }
    failure {
      echo "Pipeline failed. Check console output: ${env.BUILD_URL}console"
    }
    always {
      echo "Cleaning up workspace (optional)"
      // do not forcibly delete workspace; uncomment if desired
      // deleteDir()
    }
  }
}
