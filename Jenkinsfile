pipeline {
  agent any
  stages {
    stage('BUILD') {
      post {
        failure {
          echo 'This build has failed. See logs for details.'

        }

      }
      parallel {
        stage('Express Image') {
          steps {
            sh 'docker build -f express-image/Dockerfile             -t nodeapp-dev:trunk .'
          }
        }
        stage('Test-Unit Image') {
          steps {
            sh 'docker build -f test-image/Dockerfile             -t test-image:latest .'
          }
        }
      }
    }
    stage('TEST') {
      post {
        success {
          echo 'Build succeeded.'

        }

        unstable {
          echo 'This build returned an unstable status.'

        }

        failure {
          echo 'This build has failed. See logs for details.'

        }

      }
      parallel {
        stage('Mocha Tests') {
          steps {
            sh 'docker run --name nodeapp-dev --network="bridge" -d             -p 9000:9000 nodeapp-dev:trunk'
            sh 'docker run --name test-image -v "$PWD":/junit --network="bridge"             --link=nodeapp-dev -d -p 9001:9000             test-image:latest'
          }
        }
        stage('Quality Tests') {
          steps {
            sh 'docker login --username $DOCKER_USR --password $DOCKER_PSW'
            sh 'docker tag nodeapp-dev:trunk yjching/nodeapp-dev:latest'
            sh 'docker push yjching/nodeapp-dev:latest'
          }
        }
      }
    }
    stage('DEPLOY') {
      when {
        branch 'master'
      }
      post {
        failure {
          sh 'docker stop nodeapp-dev test-image'
          sh 'docker system prune -f'
          deleteDir()

        }

      }
      steps {
        retry(count: 3) {
          timeout(time: 10, unit: 'MINUTES') {
            sh 'docker tag nodeapp-dev:trunk yjching/nodeapp-prod:latest'
            sh 'docker push yjching/nodeapp-prod:latest'
            sh 'docker save yjching/nodeapp-prod:latest | gzip > nodeapp-prod-golden.tar.gz'
          }

        }

      }
    }
    stage('REPORTS') {
      steps {
        junit 'reports.xml'
        archiveArtifacts(artifacts: 'reports.xml', allowEmptyArchive: true)
        archiveArtifacts(artifacts: 'nodeapp-prod-golden.tar.gz', allowEmptyArchive: true)
      }
    }
    stage('CLEAN-UP') {
      steps {
        sh 'docker stop nodeapp-dev test-image'
        sh 'docker system prune -f'
        deleteDir()
      }
    }
  }
  environment {
    DOCKER = credentials('docker-hub')
  }
}