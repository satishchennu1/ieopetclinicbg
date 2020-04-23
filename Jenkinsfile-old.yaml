node("maven") {
          timeout(time: 20,unit: 'MINUTES'){
            stage('checkout code from git'){ // for display purposes
              // Get some code from a GitHub repository
              git url: "https://github.com/rajvaranasi/ieopetclinicbg.git", branch: "master" 
            }//checkout code stage
            
            stage('Build Image'){
              openshift.withCluster(){
                openshift.withProject(ieopetclinic-web-dev){
                  echo "Current Pipeline is in Build Image"
                  sh "oc start-build ieopetclinic --from-dir . --follow"  
                }
              }
            }
            stage('Code Coverage using Jacoco') { //
              archive 'target/*.jar'
              step([$class: 'JacocoPublisher', execPattern: '**/target/jacoco.exec'])
             }
            
            stage("Deploy to DEVEnvironment") {
              openshift.withCluster(ieopetclinic-web-dev){
                openshift.withProject() {
                  def dc = openshift.selector('dc', "ieopetclinic")
                  dc.rollout().latest()
                  timeout(10) {
                  dc.rollout().status()
                 }
                }
              }
            }          
          }
        }
