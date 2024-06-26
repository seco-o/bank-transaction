
def DOCKER_REGISTRY_USER = 'src32'
def AMD64_LOCAL_IMAGE = 'ibm-bank-transaction'
def SERVICE = 'bank-transaction'
def TAG = 'v0.1'
def ENVIRONMENT = 'production'

node {
  try {
    stage('Checkout') {
      sh "echo ${DOCKER_REGISTRY_USER}"
      sh "echo ${AMD64_LOCAL_IMAGE}"
      sh "echo ${TAG}"
      checkout scm
    }

    stage('compile') {
      withMaven {
        sh 'mvn clean install'
      }

      dir('target') {
        sh "cp ${SERVICE}-1.0-SNAPSHOT.jar ../build"
      }
    }

    stage('Build amd64 image') {
      dir('build') {
        sh "docker build -t ${DOCKER_REGISTRY_USER}/${AMD64_LOCAL_IMAGE}:${TAG} --platform=linux/amd64 ."
      }
    }

    stage('Push image') {
      docker.withRegistry('', 'docker-src32-accesstoken') {
        sh "docker push ${DOCKER_REGISTRY_USER}/${AMD64_LOCAL_IMAGE}:${TAG}"
      }
    }

        /** Start ---- Pre-integration Tests --------------- */
    /*stage('Pre-integration Tests') {
      parallel(

                'Quality Tests': {
                    //  sh "docker run --rm portal:v0.3-amd64"
                    sh 'podman run --publish-all --rm --name portal -d portal:v0.3-s390x'

                       sh 'chmod +x ./build/scripts/test.sh'
                       sh './build/scripts/test.sh'
                },
                'Unit Tests': {
                  sh 'chmod +x ./build/scripts/test.sh'
                  sh './build/scripts/test.sh'
                }
            )
    }*/

    stage('Deploy Image') {
      if (ENVIRONMENT == 'production') {
        timeout(time: 2, unit: 'HOURS') {
          input message: 'Approve Deploy?', ok: 'Yes'
          echo 'Your request to deploy this release to poduction is approved'
          dir('deploy') {
          //
          }
        }
      }
    }
     } finally {
    stage('cleanup') {
      echo 'Clean up workspace'
        deleteDir()
    // sh 'docker volume rm $(docker volume ls -qf dangling=true)'
    }
  }
}
