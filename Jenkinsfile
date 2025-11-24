// Jenkinsfile — Full Windows-friendly declarative pipeline for Python CI
// - venv, pip install
// - pytest with coverage (XML + HTML)
// - publish JUnit results, publish HTML coverage (HTML Publisher plugin required)
// - archive reports
// - optional Docker build & push (controlled by parameters, uses credentialsId 'docker-hub-cred')

pipeline {
  agent any

  parameters {
    booleanParam(name: 'RUN_DOCKER', defaultValue: false, description: 'Build Docker image?')
    booleanParam(name: 'PUSH_DOCKER', defaultValue: false, description: 'Push Docker image to registry (requires credentials)')
    string(name: 'DOCKER_IMAGE', defaultValue: 'ashishmondal420/python-ci-image', description: 'Docker image name (repo/name)')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        echo "Checked out: ${env.GIT_COMMIT ?: 'no-commit-info'}"
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
        // Use literal 'reports' path so Windows bat will create files in the expected place
        bat '''.venv\\Scripts\\python.exe -m pytest --junitxml=reports\\junit.xml --cov=src --cov-report=xml:reports\\coverage.xml --cov-report=html:reports\\coverage_html -q'''
      }
    }

    stage('Publish Results') {
      steps {
        echo 'Publishing test results and archiving reports'
        // Ensure reports directory exists
        bat 'if not exist reports mkdir reports'

        // Publish JUnit results
        junit 'reports\\junit.xml'

        // Publish HTML coverage (requires HTML Publisher plugin)
        publishHTML([
          allowMissing: false,
          alwaysLinkToLastBuild: true,
          keepAll: true,
          reportDir: 'reports/coverage_html',
          reportFiles: 'index.html',
          reportName: 'Coverage Report'
        ])

        // Archive reports so they can be downloaded
        archiveArtifacts artifacts: 'reports/**', fingerprint: true
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
          // Build the Docker image
          bat "docker build -t ${imageTag} ."

          if (params.PUSH_DOCKER) {
            echo 'Pushing Docker image to registry'
            // Use credential 'docker-hub-cred' (username/password) stored in Jenkins
            withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
              // Login and push (Windows bat uses %VAR% style)
              bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
              bat "docker push ${imageTag}"
            }
          } else {
            echo 'PUSH_DOCKER is false — skipping push'
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
      echo 'Pipeline finished — workspace retained (deleteDir() commented out)'
      // If you want to clean workspace uncomment the line below
      // deleteDir()
    }
  }
}
