pipeline {
  agent {
    docker {
      image 'python:3.11'
      args '-u root:root'   // run as root so installing/venv works
    }
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        echo 'Creating virtual environment and installing dependencies...'
        sh '''
          python -m venv venv
          . venv/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Test') {
      steps {
        echo 'Running unit tests...'
        sh '''
          . venv/bin/activate
          python -m unittest discover -s .
        '''
      }
    }

    stage('Deploy') {
      steps {
        echo 'Deploying application to workspace/python-app-deploy'
        sh '''
          mkdir -p ${WORKSPACE}/python-app-deploy
          cp ${WORKSPACE}/app.py ${WORKSPACE}/python-app-deploy/
        '''
      }
    }

    stage('Run Application') {
      steps {
        echo 'Starting application in background'
        sh '''
          nohup ${WORKSPACE}/venv/bin/python ${WORKSPACE}/python-app-deploy/app.py > ${WORKSPACE}/python-app-deploy/app.log 2>&1 &
          echo $! > ${WORKSPACE}/python-app-deploy/app.pid
        '''
      }
    }

    stage('Test Application') {
      steps {
        echo 'Testing running app via unit test (or curl)'
        sh '''
          # small wait so app listens
          sleep 2
          python -m unittest discover -s .
        '''
      }
    }
  }

  post {
    success { echo 'Pipeline completed successfully!' }
    failure { echo 'Pipeline failed. Inspect console logs.' }
  }
}
