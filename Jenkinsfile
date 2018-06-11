pipeline {
  agent any
  options {
    disableConcurrentBuilds()
  }
  environment {
    UNIQ_TAG = ${GIT_BRANCH.toLowerCase()}-${BUILD_NUMBER}
    STAGE_REDIS_HOST = '35.156.31.64'
    STAGE_REDIS_PORT = '6379'
    PROD_REDIS_HOST = '18.184.113.138'
    PROD_REDIS_PORT = '6379'
  }
  stages {
    stage('Build') {
      agent {
        docker {
          image 'maven:3-alpine'
          args '-v /root/.m2:/root/.m2'
        }
      }
      steps {
        sh 'mvn clean package'
        archiveArtifacts(artifacts: 'target/clickCount.war', caseSensitive: true)
      }
    }
    stage('Package') {
      steps {
        sh 'cp -f ${JENKINS_HOME}/jobs/click-count/branches/${GIT_BRANCH}/builds/${BUILD_NUMBER}/archive/target/*.war docker/runtime/'
        sh 'docker build -f docker/runtime/Dockerfile -t click-count-${UNIQ_TAG} docker/runtime'
        sh 'rm -f docker/runtime/*.war'
      }
    }
    stage('Staging') {
      steps {
        sh 'docker run -d -e XEBIA_CLICK_COUNT_REDIS_HOST=${STAGE_REDIS_HOST} -e XEBIA_CLICK_COUNT_REDIS_PORT=${STAGE_REDIS_PORT}  --name click-count-test-${UNIQ_TAG} click-count-${UNIQ_TAG}'
        sleep 5
      }
    }
    stage('Staging test') {
      steps {
        dir(path: 'APITest') {
          sh 'docker build -t click-count-api-test-${UNIQ_TAG} .'
        }

        sh '''
          STAGE_IP=`docker inspect -f "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" click-count-test-${UNIQ_TAG}`
          docker run --rm --name click-count-api-test-${UNIQ_TAG} --env WEBAPP_ADDR="http://${STAGE_IP}" --env WEBAPP_PORT=8080 click-count-api-test-${UNIQ_TAG}
        '''
      }
    }
    stage('Production') {
      when {
        branch 'master'
      }
      steps {
        sh 'set +e; docker stop click-count; docker rm click-count; set -e'
        sh 'docker run -d -p 80:8080 -e XEBIA_CLICK_COUNT_REDIS_HOST=${PROD_REDIS_HOST} -e XEBIA_CLICK_COUNT_REDIS_PORT=${PROD_REDIS_PORT} --name click-count click-count-${UNIQ_TAG}'
      }
    }
  }
  post {
    always {
        sh 'set +e; docker stop click-count-test-${UNIQ_TAG}; docker rm click-count-test-${UNIQ_TAG}; set -e'
    }
  }
}