def OCP_S390X_TOKEN = '<sha256~zdCX5-Lo1dhQyqu5ZsTOwRdc-QQzzyjH-c0Qcdw>'
def OCP_S390X_API_SERVER = '<https://c107.s390x.containers.ihost.local.com:6443>'
// oc login --token=${OCP_S390X_TOKEN} --server=${OCP_S390X_API_SERVER}
def OCP_AMD64_TOKEN = '<sha256~7dylK833xvHzms9VTl1-Ssj8MOqAEZ-HLzFb>'
def OCP_AMD64_API_SERVER = '<https://c107.amd64.containers.cloud.ibm.com:6443>'
//oc login --token=${OCP_AMD64_TOKEN} --server=${OCP_AMD64_API_SERVER}
def DOCKER_REGISTRY= 'docker.io'
def DOCKER_REGISTRY_USER = '<docker-registry-user>'
def DOCKER_REGISTRY_TOKEN = '<hQyqu5ZsTOwRdcLo1d-QQzzyjH-zdCX5>'
// docker login -u=${DOCKER_REGISTRY_USER} -p=${DOCKER_REGISTRY_TOKEN}'

def S390X_LOCAL_IMAGE = 'transaction:v0.3-s390x'
def AMD64_LOCAL_IMAGE = 'transaction:v0.3-amd64'
def TAG = 'v0.3'

def createOCPNamespaceIfNotExist(String nameSpace) {
       echo "passing variable $nameSpace"
                 try {
                        sh "oc new-project $nameSpace"
                        echo 'Succeeded!'
                      } catch (err) {
                        echo "The project you are trying to create is already exist: $nameSpace"
                        //echo "Failed: ${err}"
                     }
           sh "oc project $nameSpace"
  
  }

 def deleteOCPDeploymentSVCRouteIfExist() {
       
         try {
                sh "oc delete deployment transaction-app" 
                sh "oc delete svc transaction" 
                sh "oc delete route transaction" 
                        echo 'Succeeded!'
                      } catch (err) {
                         echo " the resource you are trying to delete doest not exist: transaction"
                       //echo "Failed: ${err}"
                     }
     
  
  }
node('workers'){
 try {
    stage('Checkout'){
     sh "echo ${OCP_S390X_TOKEN}"
     sh "echo ${OCP_AMD64_TOKEN}"
     sh "echo ${OCP_S390X_API_SERVER}"
     sh "echo ${OCP_AMD64_API_SERVER}"
     sh "echo ${DOCKER_REGISTRY_USER}"
     sh "echo ${DOCKER_REGISTRY_TOKEN}"
     sh "echo ${S390X_LOCAL_IMAGE}"
     sh "echo ${AMD64_LOCAL_IMAGE}"
     sh "echo ${TAG}"
     sh "echo ${DOCKER_REGISTRY}"
     checkout scm
   }
    stage('compile') {
          withMaven {
           sh 'mvn clean install'
      
    } 
             
   dir('target') {
                    sh "cp bank-transaction-1.0-SNAPSHOT.jar ../build"
                  
                }
          
        } 
    

    stage('Build s390x image'){
     dir('build') {

              sh "podman build -t ${S390X_LOCAL_IMAGE} --arch=s390x ."   
     
     }
    
    // sh "operator-sdk build ${S390X_LOCAL_IMAGE} --image-build-args='--arch=s390x'"
        }
     
        stage('Build amd64 image'){
          dir('build') {
           sh "podman build -t ${AMD64_LOCAL_IMAGE} --arch=amd64 ."   
           
          }
        // sh "operator-sdk build ${AMD64_LOCAL_IMAGE} --image-build-args='--arch=amd64'"
    }


       stage('create manifest'){
       
        sh "podman manifest create transaction:${TAG}"
       }
       stage('add manifest list'){
       
         sh "podman manifest add transaction:${TAG} containers-storage:localhost/${AMD64_LOCAL_IMAGE}"
         sh "podman manifest add transaction:${TAG}  containers-storage:localhost/${S390X_LOCAL_IMAGE}"
      }
       stage('Push multi-arch image'){

        sh "docker login -u=${DOCKER_REGISTRY_USER} -p=${DOCKER_REGISTRY_TOKEN} "
        
        // please uncomment this line if image exists in your docker container registry.
       // sh "skopeo delete docker://${DOCKER_REGISTRY}/${DOCKER_REGISTRY_USER}/transaction:${TAG} "  //delete the image if it exists.
        sh "podman manifest push transaction:${TAG} docker://${DOCKER_REGISTRY}/${DOCKER_REGISTRY_USER}/transaction:${TAG}" 

       }
        
        /** Start ---- Pre-integration Tests --------------- */
    stage('Pre-integration Tests'){
            parallel(
             
                'Quality Tests': {
                                 
                      //  sh "docker run --rm transaction:v0.3-amd64"
                    sh "podman run --publish-all --rm --name transaction -d transaction:v0.3-s390x"
                     
                       sh 'chmod +x ./build/scripts/test.sh'
                       sh './build/scripts/test.sh'
                                
                },
                'Unit Tests': {

                  sh 'chmod +x ./build/scripts/test.sh'   
                  sh './build/scripts/test.sh'
                 }
            )
        }
    
     /** END ---- Pre-integration Tests --------------- */
   
       stage('Deploy Image to OpenShift on x86 and s390x') {
        
         if(env.BRANCH_NAME == 'development' || env.BRANCH_NAME == 'preproduction'){
         
              dir('deploy') {
               echo 'Deploying to development or pre-production on S390X environment'
               sh "oc login --insecure-skip-tls-verify --token=${OCP_S390X_TOKEN} --server=${OCP_S390X_API_SERVER}"
               createOCPNamespaceIfNotExist('staging-deployment')
                             
                deleteOCPDeploymentSVCRouteIfExist()
                sh "oc create -f bank-transaction.yaml" // & deployment config
                sh "oc expose service transaction" // Push new stream
               
                 /** IBM CLOUD - X86 ENVIRONMENT */
               echo 'Deploying to development or pre-production on IBM Cloud on X86 environment'
               sh "oc login --insecure-skip-tls-verify --token=${OCP_AMD64_TOKEN} --server=${OCP_AMD64_API_SERVER}"
                createOCPNamespaceIfNotExist('staging-deployment')
                     
                deleteOCPDeploymentSVCRouteIfExist()
                sh "oc create -f bank-transaction.yaml" // & deployment config
                sh "oc expose service transaction" // Push new stream
               
               }  
          }
        if(env.BRANCH_NAME == 'master'){
           timeout(time: 2, unit: "HOURS") {
            input message: "Approve Deploy?", ok: "Yes"
            echo 'Your request to deploy this release to poduction is approved'
            dir('deploy') {
             
                /** LINUXONE/ONPREM - S390X ENVIRONMENT */
                sh "oc login --insecure-skip-tls-verify --token=${OCP_S390X_TOKEN} --server=${OCP_S390X_API_SERVER}"
                 createOCPNamespaceIfNotExist('production-deployment')   
               
                deleteOCPDeploymentSVCRouteIfExist()
                sh "oc create -f bank-transaction-prod.yaml" // & deployment config
                sh "oc expose service transaction" // Push new stream
             
             
               /** IBM CLOUD - X86 ENVIRONMENT */

               sh "oc login --insecure-skip-tls-verify --token=${OCP_AMD64_TOKEN} --server=${OCP_AMD64_API_SERVER}"
               createOCPNamespaceIfNotExist('production-deployment')
                   
                deleteOCPDeploymentSVCRouteIfExist()
                sh "oc create -f bank-transaction-prod.yaml" // & deployment config
                sh "oc expose service transaction" // Push new stream

               } 
          }
                   
    } 
 }
  
   
    
     } finally {
    stage('cleanup') {
      echo 'Clean up workspace'
        deleteDir()
      echo 'remove all images'
      sh 'docker image prune -a -f'
      sh "docker stop transaction"
     // sh 'docker volume rm $(docker volume ls -qf dangling=true)'
    }
  }
 }

