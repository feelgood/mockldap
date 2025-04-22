@Library('feelgood-scripts')_

pipeline {
  agent {
    docker {
      image 'feelgoodsvenska/jenkins-build:python'
      args '-v /home/jenkins/.cache:/home/jenkins/.cache '
    }
  }

  stages {
    stage('Run linter') {
      steps {
        sh '''
        mkdir test-reports && touch test-reports/flake8.txt
        flake8 --exit-zero --output-file=test-reports/flake8.txt --config /home/jenkins/.config/flake8
        '''
      }
    }

    stage('Install dependencies') {
      steps {
        sh '''
        python3 -m venv venv
        . venv/bin/activate
        pip install --upgrade pip
        pip install -r requirements.txt'''
      }
    }
    stage('Run unit tests') {
      steps {
        sh '''. venv/bin/activate
        PYTHONPATH=. pytest . --junit-xml=test-reports/junit/junit.xml'''
      }
    }
    stage('Build and publish dist') {
      when {
        buildingTag()
      }
      steps {
        uploadPythonToProget()
      }
    }
  }
  post {
    always {
      junit(testResults: 'test-reports/**/*.xml', allowEmptyResults: true)
      recordIssues(
        tool: flake8(pattern: 'test-reports/flake8.txt'),
        enabledForFailure: true,
        qualityGates: [[type: 'TOTAL', unstable: true, threshold: 1]]
      )
      slackBuildStatus(currentBuild.currentResult)
      deleteDir()
    }
  }
}
