pipeline {
   agent any

   options {
        // set a timeout of 20 minutes for this pipeline
        timeout(time: 20, unit: 'MINUTES')
    } //options

    environment {
        APP_NAME    = "ieopetclinic"
        GIT_REPO    = "https://github.com/rajvaranasi/ieopetclinic.git"
        GIT_BRANCH  = "master"

        CICD_PRJ    = "ieopetclinic-web-build"
        CICD_DEV    = "ieopetclinic-web-dev"
        CICD_UAT   = "ieopetclinic-web-uat"
        CICD_PREPROD  = "ieopetclinic-web-preprod"
        SVC_PORT    = 8080
    } //environment
    
    node("maven"){
        stage('checkout code from git'){
            git url: "https://github.com/rajvaranasi/petspringdocker.git", branch: "master"    
         }

        stage('Build Image') {
          openshift.withCluster(){
            openshift.withProject($CICD_DEV){
              sh "oc start-build ieopetclinic --from-dir . --follow "
            } //withProject
          }//withCluster
        }
    }//node(maven)
} //pipeline 
