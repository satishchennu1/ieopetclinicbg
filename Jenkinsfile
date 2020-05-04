pipeline {
    tools{
        maven 'MAVEN'
    }
    agent any 
    options {
        // set a timeout of 20 min
        //utes for this pipeline
        timeout(time: 20, unit: 'MINUTES')
    } //options
    environment {
        APP_NAME    = "ieopetclinic"
        GIT_REPO    = "https://github.com/rajvaranasi/ieopetclinicbg.git"
        GIT_BRANCH  = "master"
        UAT_ROUTE_URL = "ieopetclinic-uat"
        PREPROD_ROUTE_URL = "ieopetclinic-preprod"
        def tag="blue"
        def altTag="green"
        def verbose="false"

        CICD_PRJ    = "ieopetclinic-bluegreen-cicd"
        CICD_DEV    = "ieopetclinic-bluegreen-dev"
        CICD_UAT   =  "ieopetclinic-bluegreen-uat"
        CICD_PREPROD  = "ieopetclinic-bluegreen-preprod"
        SVC_PORT    = 8080
    } //environment
    
 
    stages {
        stage('Git CheckOut') {

                steps {
                    echo "Testing if 'Service' resource is operational and responding"
                    script {
                        openshift.withCluster() {
                                openshift.withProject() {
                                   checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/rajvaranasi/ieopetclinicbg.git']]])
                                } // withProject
                        } // withCluster
                    } // script
                } // steps
            } //stage 
        stage('clean workspace') {

                steps {
                    script {
                        openshift.withCluster() {
                                openshift.withProject() {
                                    echo "-=-Cleaning workspace-=-"
                                    sh "mvn clean "
                                } // withProject
                        } // withCluster
                    } // script
                } // steps
            } //stage

       stage('Unit Test') {

                steps {
                    script {
                        openshift.withCluster() {
                                openshift.withProject() {
                                    echo "---Unit Test---"
                                    sh "mvn test"
                                    step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
                                } // withProject
                        } // withCluster
                    } // script
                } // steps
            } //stage
      stage('Integration Test') {

                steps {
                    echo "Testing if 'Service' resource is operational and responding"
                    script {
                        openshift.withCluster() {
                                openshift.withProject() {
                                    echo "--- IntegrationTest---"
                                    sh "mvn verify"
                                } // withProject
                        } // withCluster
                    } // script
                } // steps
            } //stage
        stage('Code Coverage using jacoco') {
                steps {
                    echo "Code Coverage using jacoco"
                    script {
                        openshift.withCluster() {
                                openshift.withProject() {
                                    archive 'target/*.jar'
                                    step([$class: 'JacocoPublisher', execPattern: '**/target/jacoco.exec'])
                                } // withProject
                        } // withCluster
                    } // script
                } // steps
            } //stage

        stage('Build Image ') {
                steps {
                    echo "Sample Build stage using project ${CICD_DEV}"
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${CICD_DEV}")
                            {

                                if (openshift.selector("bc",APP_NAME).exists()) {
                                    echo "Using existing BuildConfig. Running new Build"
                                    def bc = openshift.startBuild(APP_NAME)
                                    openshift.set("env dc/${APP_NAME} BUILD_NUMBER=${BUILD_NUMBER}")
                                    // output build logs to the Jenkins conosole
                                    echo "Logs from build"
                                    def result = bc.logs('-f')
                                    // actions that took place
                                    echo "The logs operation require ${result.actions.size()} 'oc' interactions"
                                    // see exactly what oc command was executed.
                                    echo "Logs executed: ${result.actions[0].cmd}"
                                } else {
                                    echo "No proevious BuildConfig. Creating new BuildConfig."
                                    def myNewApp = openshift.newApp (
                                        "${GIT_REPO}#${GIT_BRANCH}", 
                                        "--name=${APP_NAME}", 
                                        "--context-dir=${CONTEXT_DIR}", 
                                        "-e BUILD_NUMBER=${BUILD_NUMBER}", 
                                        "-e BUILD_ENV=${openshift.project()}"
                                        )
                                    echo "new-app myNewApp ${myNewApp.count()} objects named: ${myNewApp.names()}"
                                    myNewApp.describe()
                                    // selects the build config 
                                    def bc = myNewApp.narrow('bc')
                                    // output build logs to the Jenkins conosole
                                    echo "Logs from build"
                                    def result = bc.logs('-f')
                                    // actions that took place
                                    echo "The logs operation require ${result.actions.size()} 'oc' interactions"
                                    // see exactly what oc command was executed.
                                    echo "Logs executed: ${result.actions[0].cmd}"
                                } //else

                                echo "Tag Container image with 'build number' as version"
                                openshift.tag("${APP_NAME}:latest", "${APP_NAME}:v${BUILD_NUMBER}")

                                echo "Validating Route for Service exist, if Not create Route"
                                if (!openshift.selector("route",APP_NAME).exists()) {
                                    openshift.selector("svc",APP_NAME).expose()
                                }

                            } // project
                        } // cluster
                    } // script
                } // steps
            } //stage-build
        
        stage('Deploy to DEV Project') {
                steps {
                    echo "Sample Build stage using project ${CICD_DEV}"
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${CICD_DEV}")
                            {
                                sleep 5
                                def dc = openshift.selector("dc", "ieopetclinic")
                                dc.rollout().status()                                
                            } // project
                        } // cluster
                    } // script
                } // steps
            } //stage-build
        stage('Smoke Test DEVURL') {

                steps {
                    echo "Testing if 'Service' resource is operational and responding"
                    script {
                        openshift.withCluster() {
                                openshift.withProject("${CICD_DEV}") {
                                   sleep 30 
                                   //echo sh (script: 'curl -I http://ieopetclinic-ieopetclinic-bluegreen-dev.apps.ocp43.itblab.uspto.gov', returnStdout: true)
                                    sh script: '''response=$(curl --fail -s -o /dev/null -w "%{http_code}\\n" http://ieopetclinic-ieopetclinic-bluegreen-dev.apps.ocp43.itblab.uspto.gov)
                                    if [ "$response" -ne 200 ]
                                    then
                                      exit 1
                                    fi'''
                                    
                                } // withProject
                        } // withCluster
                    } // script
                } // steps
            } //stage 


        stage('approval Message to deploy in UAT Controlled Environment') {

                steps {
                    echo "Verify if all Functional Test are passed in DEV and QA certifies the Build"
                    script {
                        openshift.withCluster() {
                                openshift.withProject("${CICD_DEV}") {
                                  input message: "Need approval to move to UAT Controlled environment: Approve?", id: "approval"
                                   
                                } // withProject
                        } // withCluster
                    } // script
                } // steps
            } //stage


        
        stage('Promote to UAT') {
                steps {
                    echo "Sample usage of UAT "
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${CICD_UAT}") {
                                sh "oc project ${CICD_UAT}"
                                sh "oc get route"
                                sh "oc get route ${UAT_ROUTE_URL} -o jsonpath='{ .spec.to.name }'> activeservice"
                                activeService = readFile('activeservice').trim()
                                if (activeService == "${APP_NAME}-blue-uat") {
                                        tag = "green"
                                        altTag = "blue"
                                    }//active-service
                                sh "oc get route ${APP_NAME}-${tag}-uat -o jsonpath='{ .spec.host }' > routehost"
                                //openshift.tag("${CICD_DEV}/${APP_NAME}:v${BUILD_NUMBER}", "${CICD_UAT}/${APP_NAME}:v${BUILD_NUMBER}")
                                openshift.tag("${CICD_DEV}/${APP_NAME}:latest", "${CICD_UAT}/${APP_NAME}-uat:latest")
                                openshift.tag("${CICD_UAT}/${APP_NAME}-uat:latest", "${CICD_UAT}/${APP_NAME}-${tag}-uat:latest")
                                sleep 10
                                def dc = openshift.selector('dc', "${APP_NAME}-${tag}-uat")
                                dc.rollout().status()
                                sleep 20
                            }
                        }
                    } // script
                } //steps 
            } //stage
        stage('Switch over Live Traffic to New Version in UAT') {
                steps {
                    echo "deploying and routing the traffic  ${CICD_UAT}"
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${CICD_UAT}")
                            {   
                                sh "oc get route ${UAT_ROUTE_URL} -o jsonpath='{ .spec.to.name }'> activeservice"
                                activeService = readFile('activeservice').trim()
                                if (activeService == "${APP_NAME}-blue-uat") {
                                        tag = "green"
                                        altTag = "blue"
                                    }//active-service
                                sh "oc get route ${APP_NAME}-${tag}-uat -o jsonpath='{ .spec.host }' > routehost"
                                sh "oc set -n ${CICD_UAT} route-backends ${UAT_ROUTE_URL} ${APP_NAME}-${tag}-uat=100 ${APP_NAME}-${altTag}-uat=0"
                                sh "oc get routes"
                            } // project
                        } // cluster
                    } // script
                } // steps
            } //stage-build
        stage('Smoke Test UAT-URL') {

                steps {
                    echo "Testing if 'Service' resource is operational and responding"
                    script {
                        openshift.withCluster() {
                                openshift.withProject("${CICD_UAT}") {
                                   sleep 30
                                    sh script: '''response=$(curl --fail -s -o /dev/null -w "%{http_code}\\n" http://ieopetclinic-uat-ieopetclinic-bluegreen-uat.apps.ocp43.itblab.uspto.gov)
                                    if [ "$response" -ne 200 ]
                                    then
                                      exit 1
                                    fi'''

                                } // withProject
                        } // withCluster
                    } // script
                } // steps
            } //stage

         stage('Need approval Message to deploy in PREPROD Controlled Environment') {

                steps {
                    echo "Verify if all Functional Test are passed in UAT  and QA certifies the Build"
                    script {
                        openshift.withCluster() {
                                openshift.withProject("${CICD_UAT}") {
                                  input message: "Need approval to move to PREPROD Controlled environment: Approve?", id: "approval"

                                } // withProject
                        } // withCluster
                    } // script
                } // steps
            } //stage

        stage('Promote to PREPROD') {
                steps {
                    echo "Trying to Deploy APP to PREPROD "
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${CICD_PREPROD}") {
                                sh "oc project ${CICD_PREPROD}"
                                echo "routes before the deployment "
                                sh "oc get route"
                                sh "oc get route ${PREPROD_ROUTE_URL} -o jsonpath='{ .spec.to.name }'> activeservice"
                                activeService = readFile('activeservice').trim()
                                if (activeService == "${APP_NAME}-blue-preprod") {
                                        tag = "green"
                                        altTag = "blue"
                                    }//active-service
                                sh "oc get route ${APP_NAME}-${tag}-preprod -o jsonpath='{ .spec.host }' > routehost"
                                //openshift.tag("${CICD_DEV}/${APP_NAME}:v${BUILD_NUMBER}", "${CICD_UAT}/${APP_NAME}:v${BUILD_NUMBER}")
                                openshift.tag("${CICD_UAT}/${APP_NAME}-uat:latest", "${CICD_PREPROD}/${APP_NAME}-preprod:latest")
                                openshift.tag("${CICD_PREPROD}/${APP_NAME}-preprod:latest", "${CICD_PREPROD}/${APP_NAME}-${tag}-preprod:latest")
                                sleep 10
                                def dc = openshift.selector('dc', "${APP_NAME}-${tag}-preprod")
                                dc.rollout().status()
                                sleep 20
                                
                                } //project
                           } //cluster 
                    } // script
                } //steps 
            } //stage
        stage('Switch over live traffic to  New Version in PREPROD') {
                steps {
                    echo "deploying and routing the traffic  ${CICD_PREPROD}"
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${CICD_PREPROD}")
                            {   
                                sh "oc get route ${PREPROD_ROUTE_URL} -o jsonpath='{ .spec.to.name }'> activeservice"
                                activeService = readFile('activeservice').trim()
                                if (activeService == "${APP_NAME}-blue-preprod") {
                                        tag = "green"
                                        altTag = "blue"
                                    }//active-service
                                sh "oc get route ${APP_NAME}-${tag}-preprod -o jsonpath='{ .spec.host }' > routehost"
                                sh "oc set -n ${CICD_PREPROD} route-backends ${PREPROD_ROUTE_URL} ${APP_NAME}-${tag}-preprod=100 ${APP_NAME}-${altTag}-preprod=0"
                                sh "oc get routes"
                                
                                
                                
                            } // project
                        } // cluster
                    } // script
                } // steps
            } //stage-build
        stage('Smoke Test PREPROD-URL') {

                steps {
                    echo "Testing if 'Service' resource is operational and responding"
                    script {
                        openshift.withCluster() {
                                openshift.withProject("${CICD_DEV}") {
                                   sleep 30
                                    sh script: '''response=$(curl --fail -s -o /dev/null -w "%{http_code}\\n" ieopetclinic-preprod-ieopetclinic-bluegreen-preprod.apps.ocp43.itblab.uspto.gov)
                                    if [ "$response" -ne 200 ]
                                    then
                                      exit 1
                                    fi'''

                                } // withProject
                        } // withCluster
                    } // script
                } // steps
            } //stage


        }
 }
