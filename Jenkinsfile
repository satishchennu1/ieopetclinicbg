node("maven") {
          timeout(time: 20,unit: 'MINUTES'){
            stage('checkout code from git'){ // for display purposes
              // Get some code from a GitHub repository
              git url: "https://github.com/rajvaranasi/petspringdocker.git", branch: "master" 
            }//checkout code stage
            stage("Clean WorkSpace") {
                sh "mvn clean"
              }//building the war file
            stage('Build Image'){
              openshift.withCluster(){
                openshift.withProject(){
                  sh "oc start-build ieopetclinic --from-dir . --follow"  
                }
              }
            }
            stage('Code Coverage using Jacoco') { //
              archive 'target/*.jar'
              step([$class: 'JacocoPublisher', execPattern: '**/target/jacoco.exec'])
             }
            stage("Deploy to DeveloperSandBox") {
              openshift.withCluster() {
                openshift.withProject() {
                  def dc = openshift.selector('dc', "ieopetclinic")
                  dc.rollout().status()
                }
              }
            }
           
            stage("approval Message to deploy in DEV") {
              input message: "Need approval to move to DEV environment: Approve?", id: "approval"
            }

            stage('Tag Images') {
              openshift.withCluster() {
                openshift.withProject(){
                  openshift.tag("ieopetclinic:latest", "ieopetclinic:dev")
                }
              }
            }
            stage('Promote to DEV'){
              openshift.withCluster(){
                openshift.withProject(){
                  openshift.tag("ieopetclinic:latest", "ieopetclinic:dev")
                  openshift.selector("dc","ieopetclinic-dev").rollout().status()
                }
              }
            }
            stage("approval Message to deploy in UAT") {
              input message: "Need approval to move to Controlled UAT Environment: Approve?", id: "approval"
            }          
            stage('Promote to UAT'){
              openshift.withCluster(){
                openshift.withProject(){
                  openshift.tag("ieopetclinic:latest", "ieopetclinic:uat")
                  openshift.selector("dc","ieopetclinic-uat").rollout().status()
                }
              }
            }


          }
        }
