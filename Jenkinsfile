pipeline {
    agent { 
        docker { 
            image 'python:3.6.9'
            args  '-u root:root' 
        } 
    }
    stages {
        stage('build') {
            steps {
                sh 'pip install --upgrade pip'
                sh 'pip install -r requirements.txt'
            }
        }
        stage('Test') {
           steps {
              sh 'rm flake8-output.txt'
               sh 'flake8  --max-line-length 99 --exit-zero --output-file flake8-output.txt'
               sh 'flake8_junit flake8-output.txt flake8-output.xml'
          }
       }
    }
    post {
      always {
        junit 'flake8-output.xml'
    }
    failure {
        echo 'Failed!'
    }
    success {
        echo 'Done!'
    }
  }
}
