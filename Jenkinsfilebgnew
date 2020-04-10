pipeline {
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
        def tag="blue"
        def altTag="green"
        def verbose="false"

        CICD_PRJ    = "ieopetclinic-web-build"
        CICD_DEV    = "ieopetclinic-web-dev"
        CICD_UAT   =  "ieopetclinic-web-uat"
        CICD_PREPROD  = "ieopetclinic-web-preprod"
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
        stage('Deploy to DEV Project') {
                steps {
                    echo "Sample Build stage using project ${CICD_DEV}"
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${CICD_DEV}")
                            {
                                sleep 30
                                openshift.selector("dc", "ieopetclinic").rollout().latest();
                                
                            } // project
                        } // cluster
                    } // script
                } // steps
            } //stage-build
        stage('Test Dev URL') {

                steps {
                    echo "Testing if 'Service' resource is operational and responding"
                    script {
                        openshift.withCluster() {
                                openshift.withProject("${CICD_DEV}") {
                                    echo sh (script: "curl -I http://${APP_NAME}.${CICD_DEV}.apps.testocpaws.ocpawscontainers.com", returnStdout: true)
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
                                sh "oc project ieopetclinic-web-uat"
                                sh "oc get route"
                                sh "oc get route ${UAT_ROUTE_URL} -o jsonpath='{ .spec.to.name }'> activeservice"
                                activeService = readFile('activeservice').trim()
                                if (activeService == "${APP_NAME}-blue") {
                                        tag = "green"
                                        altTag = "blue"
                                    }//active-service
                                sh "oc get route ${APP_NAME}-${tag} -o jsonpath='{ .spec.host }' > routehost"
                                //openshift.tag("${CICD_DEV}/${APP_NAME}:v${BUILD_NUMBER}", "${CICD_UAT}/${APP_NAME}:v${BUILD_NUMBER}")
                                openshift.tag("${CICD_DEV}/${APP_NAME}:latest", "${CICD_UAT}/${APP_NAME}:latest")
                                openshift.tag("${CICD_UAT}/${APP_NAME}:latest", "${CICD_UAT}/${APP_NAME}-${tag}:latest")
                                sleep 10
                                def dc = openshift.selector('dc', "${APP_NAME}-${tag}")
                                dc.rollout().status()
                                sleep 20
                            }
                        }
                    } // script
                } //steps 
            } //stage
        stage('Route the Live Traffic in UAT') {
                steps {
                    echo "deploying and routing the traffic  ${CICD_UAT}"
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${CICD_UAT}")
                            {   
                                sh "oc get route ${UAT_ROUTE_URL} -o jsonpath='{ .spec.to.name }'> activeservice"
                                activeService = readFile('activeservice').trim()
                                if (activeService == "${APP_NAME}-blue") {
                                        tag = "green"
                                        altTag = "blue"
                                    }//active-service
                                sh "oc get route ${APP_NAME}-${tag} -o jsonpath='{ .spec.host }' > routehost"
                                sh "oc set -n ${CICD_UAT} route-backends ${UAT_ROUTE_URL} ${APP_NAME}-${tag}=100 ${APP_NAME}-${altTag}=0"
                                sh "oc get routes"
                            } // project
                        } // cluster
                    } // script
                } // steps
            } //stage-build
        stage('Promote to PREPROD') {
                steps {
                    echo "Trying to Deploy APP to PREPROD "
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${CICD_PREPROD}") {
                                sh "oc project ${CICD_PREPROD}"
                                echo "routes before the deployment "
                                sh "oc get route"
                                sh "oc get route ${APP_NAME} -o jsonpath='{ .spec.to.name }'> activeservice"
                                activeService = readFile('activeservice').trim()
                                if (activeService == "${APP_NAME}-blue") {
                                        tag = "green"
                                        altTag = "blue"
                                    }//active-service
                                sh "oc get route ${APP_NAME}-${tag} -o jsonpath='{ .spec.host }' > routehost"
                                //openshift.tag("${CICD_DEV}/${APP_NAME}:v${BUILD_NUMBER}", "${CICD_UAT}/${APP_NAME}:v${BUILD_NUMBER}")
                                openshift.tag("${CICD_UAT}/${APP_NAME}:latest", "${CICD_PREPROD}/${APP_NAME}:latest")
                                openshift.tag("${CICD_PREPROD}/${APP_NAME}:latest", "${CICD_PREPROD}/${APP_NAME}-${tag}:latest")
                                sleep 10
                                def dc = openshift.selector('dc', "${APP_NAME}-${tag}")
                                dc.rollout().status()
                                sleep 20
                                
                                } //project
                           } //cluster 
                    } // script
                } //steps 
            } //stage
        stage('Route the Live Traffic in PREPROD') {
                steps {
                    echo "deploying and routing the traffic  ${CICD_PREPROD}"
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${CICD_PREPROD}")
                            {   
                                sh "oc get route ${APP_NAME} -o jsonpath='{ .spec.to.name }'> activeservice"
                                activeService = readFile('activeservice').trim()
                                if (activeService == "${APP_NAME}-blue") {
                                        tag = "green"
                                        altTag = "blue"
                                    }//active-service
                                sh "oc get route ${APP_NAME}-${tag} -o jsonpath='{ .spec.host }' > routehost"
                                sh "oc set -n ${CICD_PREPROD} route-backends ${APP_NAME} ${APP_NAME}-${tag}=100 ${APP_NAME}-${altTag}=0"
                                sh "oc get routes"
                                
                                
                                
                            } // project
                        } // cluster
                    } // script
                } // steps
            } //stage-build
        }
 }
