pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        echo "Checking out from GitHub"
        checkout scm
      }
    }

    stage('Setup Python') {
      steps {
        echo "Create venv and install deps"
        // create venv
        bat 'python -m venv .venv'
        // install dependencies
        bat '.venv\\Scripts\\activate && pip install --upgrade pip && pip install -r requirements.txt'
      }
    }

    stage('Run Tests') {
      steps {
        echo "Running pytest and producing JUnit XML"
        // run pytest and write JUnit output
        bat '.venv\\Scripts\\activate && pytest --junitxml=reports\\junit.xml --maxfail=1 -q'
      }
    }

    stage('Archive & Publish') {
      steps {
        echo "Publishing results and archiving artifacts"
        // make sure reports folder exists for junit step
        bat 'if not exist reports mkdir reports'
        // publish junit results
        junit 'reports\\junit.xml'
        // archive anything in dist
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
