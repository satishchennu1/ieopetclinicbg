# petspringdocker
#command to create and install the file 
oc new-app https://github.com/satishchennu1/petspringdocker.git --strategy=docker --name=petspringdocker
oc new-project mypetspringapp 

oc get templates --namespace openshift | grep "mysql-persistent" 
 
oc new-app mysql-persistent -e MYSQL_USER=petclinic -e MYSQL_PASSWORD=petclinic -e MYSQL_ROOT_PASSWORD=petclinic -e MYSQL_DATABASE=petclinic --name petclinic-mysql

oc get pods 

CREATE DATABASE IF NOT EXISTS petclinic;

ALTER DATABASE petclinic
  DEFAULT CHARACTER SET utf8
  DEFAULT COLLATE utf8_general_ci;

GRANT ALL PRIVILEGES ON petclinic.* TO pc@localhost IDENTIFIED BY 'pc';

USE petclinic;

CREATE TABLE IF NOT EXISTS vet (
  id INT(4) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(30),
  last_name VARCHAR(30),
  INDEX(last_name)
) engine=InnoDB;

CREATE TABLE IF NOT EXISTS specialty (
  id INT(4) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(80),
  INDEX(name)
) engine=InnoDB;

CREATE TABLE IF NOT EXISTS vet_specialty (
  vet INT(4) UNSIGNED NOT NULL,
  specialty INT(4) UNSIGNED NOT NULL,
  FOREIGN KEY (vet) REFERENCES vet(id),
  FOREIGN KEY (specialty) REFERENCES specialty(id),
  UNIQUE (vet,specialty)
) engine=InnoDB;

CREATE TABLE IF NOT EXISTS pet_type (
  id INT(4) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(80),
  INDEX(name)
) engine=InnoDB;

CREATE TABLE IF NOT EXISTS owner (
  id INT(4) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(30),
  last_name VARCHAR(30),
  address VARCHAR(255),
  city VARCHAR(80),
  telephone VARCHAR(20),
  INDEX(last_name)
) engine=InnoDB;

CREATE TABLE IF NOT EXISTS pet (
  id INT(4) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(30),
  birth_date DATE,
  type_id INT(4) UNSIGNED NOT NULL,
  owner_id INT(4) UNSIGNED NOT NULL,
  INDEX(name),
  FOREIGN KEY (owner_id) REFERENCES owner(id),
  FOREIGN KEY (type_id) REFERENCES pet_type(id)
) engine=InnoDB;

CREATE TABLE IF NOT EXISTS visit (
  id INT(4) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  pet_id INT(4) UNSIGNED NOT NULL,
  date DATE,
  description VARCHAR(255),
  FOREIGN KEY (pet_id) REFERENCES pet(id)
) engine=InnoDB;

oc get svc

oc new-app registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift~https://github.com/rajvaranasi/ieopetclinic.git --name ieopetclinic

environment variables in deployment config 

a) database - mysql 

b) spring.datasource.url - jdbc:mysql://172.30.186.24:3306/petclinic


c) spring.datasource.username root 

d) spring.datasource.password petclinic


oc expose svc/ieopetclinic

https://github.com/satishchennu1/openshiftv3-workshop/blob/master/12-Code-Promotion-Across-Environments.adoc

./oc.exe get is
NAME                  IMAGE REPOSITORY                                                                    TAGS     UPDATED
ieopetclinic          image-registry.openshift-image-registry.svc:5000/ieopetclinic/ieopetclinic          latest   3 hours ago
openjdk18-openshift   image-registry.openshift-image-registry.svc:5000/ieopetclinic/openjdk18-openshift   latest   5 hours ago


$ oc.exe describe is ieopetclinic
Name:                   ieopetclinic
Namespace:              ieopetclinic
Created:                5 hours ago
Labels:                 app=ieopetclinic
Annotations:            openshift.io/generated-by=OpenShiftNewApp
Docker Pull Spec:       image-registry.openshift-image-registry.svc:5000/ieopetclinic/ieopetclinic
Image Lookup:           local=false
Unique Images:          5
Tags:                   1

latest
  no spec tag

  * image-registry.openshift-image-registry.svc:5000/ieopetclinic/ieopetclinic@sha256:1993dcb4b355b7de372b857826fd275167f4b2ed977824e306885c569a119fb7
      3 hours ago
    image-registry.openshift-image-registry.svc:5000/ieopetclinic/ieopetclinic@sha256:1ef39d56e6810246f979dfa5af9bba12e28bc225a7bd3de5d9469a19debc5856
      3 hours ago
    image-registry.openshift-image-registry.svc:5000/ieopetclinic/ieopetclinic@sha256:c219eeeeb8e690f8755058954bc60f2388209bde7e181702aca023ea1acee8aa
      3 hours ago
    image-registry.openshift-image-registry.svc:5000/ieopetclinic/ieopetclinic@sha256:e9163babddf97413c0c150ffbbfd1e4062447011b2796af751fb06698f88fbca
      4 hours ago
    image-registry.openshift-image-registry.svc:5000/ieopetclinic/ieopetclinic@sha256:58a5d77c79a68c82d948f43cc93f1ce3cca4f12391777a601ac7a8349a0a7812
      5 hours ago

$ oc.exe new-app registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift~https://github.com/rajvaranasi/ieopetclinic.git --name ieopetclinic-dev
--> Found Docker image fc8de88 (5 weeks old) from registry.access.redhat.com for "registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift"

    Java Applications
    -----------------
    Platform for building and running plain Java applications (fat-jar and flat classpath)

    Tags: builder, java

    * An image stream tag will be created as "openjdk18-openshift:latest" that will track the source image
    * A source build using source code from https://github.com/rajvaranasi/ieopetclinic.git will be created
      * The resulting image will be pushed to image stream tag "ieopetclinic-dev:latest"
      * Every time "openjdk18-openshift:latest" changes a new build will be triggered
    * This image will be deployed in deployment config "ieopetclinic-dev"
    * Ports 8080/tcp, 8443/tcp, 8778/tcp will be load balanced by service "ieopetclinic-dev"
      * Other containers can access this service through the hostname "ieopetclinic-dev"

--> Creating resources ...
    imagestream.image.openshift.io "ieopetclinic-dev" created
    buildconfig.build.openshift.io "ieopetclinic-dev" created
    deploymentconfig.apps.openshift.io "ieopetclinic-dev" created
    service "ieopetclinic-dev" created
--> Success
    Build scheduled, use 'oc logs -f bc/ieopetclinic-dev' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/ieopetclinic-dev'
    Run 'oc status' to view your app.

schennu@ECCICUN6611A MINGW64 /c/workspace/oc (master)
$ oc
oc.exe        occache.dll   ocsetapi.dll  

schennu@ECCICUN6611A MINGW64 /c/workspace/oc (master)
$ oc.exe expose svc/ieopetclinic-dev
route.route.openshift.io/ieopetclinic-dev exposed

schennu@ECCICUN6611A MINGW64 /c/workspace/oc (master)
$ oc.exe new-app registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift~https://github.com/rajvaranasi/ieopetclinic.git --name ieopetclinic-uat
--> Found Docker image fc8de88 (5 weeks old) from registry.access.redhat.com for "registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift"

    Java Applications
    -----------------
    Platform for building and running plain Java applications (fat-jar and flat classpath)

    Tags: builder, java

    * An image stream tag will be created as "openjdk18-openshift:latest" that will track the source image
    * A source build using source code from https://github.com/rajvaranasi/ieopetclinic.git will be created
      * The resulting image will be pushed to image stream tag "ieopetclinic-uat:latest"
      * Every time "openjdk18-openshift:latest" changes a new build will be triggered
    * This image will be deployed in deployment config "ieopetclinic-uat"
    * Ports 8080/tcp, 8443/tcp, 8778/tcp will be load balanced by service "ieopetclinic-uat"
      * Other containers can access this service through the hostname "ieopetclinic-uat"

--> Creating resources ...
    imagestream.image.openshift.io "ieopetclinic-uat" created
    buildconfig.build.openshift.io "ieopetclinic-uat" created
    deploymentconfig.apps.openshift.io "ieopetclinic-uat" created
    service "ieopetclinic-uat" created
--> Success
    Build scheduled, use 'oc logs -f bc/ieopetclinic-uat' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/ieopetclinic-uat'
    Run 'oc status' to view your app.

schennu@ECCICUN6611A MINGW64 /c/workspace/oc (master)
$ oc.exe expose svc/ieopetclinic-uat
route.route.openshift.io/ieopetclinic-uat exposed

schennu@ECCICUN6611A MINGW64 /c/workspace/oc (master)
$ oc
oc.exe        occache.dll   ocsetapi.dll  

schennu@ECCICUN6611A MINGW64 /c/workspace/oc (master)
$ oc.exe get is
NAME                  IMAGE REPOSITORY                                                                    TAGS         UPDATED
ieopetclinic          image-registry.openshift-image-registry.svc:5000/ieopetclinic/ieopetclinic          dev,latest   12 minutes ago
ieopetclinic-dev      image-registry.openshift-image-registry.svc:5000/ieopetclinic/ieopetclinic-dev
ieopetclinic-uat      image-registry.openshift-image-registry.svc:5000/ieopetclinic/ieopetclinic-uat
openjdk18-openshift   image-registry.openshift-image-registry.svc:5000/ieopetclinic/openjdk18-openshift   latest       5 hours ago

oc.exe set resources dc ieopetclinic --requests=cpu=200m --limits=cpu=500m
oc.exe set resources dc ieopetclinic-dev --requests=cpu=200m --limits=cpu=500m
oc.exe set resources dc ieopetclinic-uat --requests=cpu=200m --limits=cpu=500m

We have set the CPU request (initial allocation) as 200 millicores and limit (maximum allocation) to 500 millicores. So when we ask HPA to scale based on percentage workload, it measures based on these numbers.

$ oc.exe autoscale dc ieopetclinic --cpu-percent=50 --min=1 --max=10
oc.exe autoscale dc ieopetclinic-dev --cpu-percent=50 --min=1 --max=10
oc.exe autoscale dc ieopetclinic-uat --cpu-percent=50 --min=1 --max=10


horizontalpodautoscaler.autoscaling/hpademo autoscaled
Here we are did two things:

cpu-percent=50 indicates that when the CPU usage (based on requests and limits) reaches 50%, HPA should spin up additional pods

--min=1 --max=10 sets upper and lower limits for the number of pods. We want to run minimum 1 pod and maximum it can scale up to 10 pods. Why? We cannot allow our application to consume all resources on the cluster.. right?

Generate Load
Now it is time to generate load and test

Open another terminal and login to the cluster. Make sure you are in the same project. And run the load generator pod from that terminal.

$ oc run --generator=run-pod/v1 -it --rm load-generator --image=busybox /bin/sh
If you don't see a command prompt, try pressing enter.
~ $
This spins up a busybox image from where we will generate the load.

Get the URL for your application oc get route hpademo --template={{.spec.host}}, and use that in the following command inside the load generator at the prompt

while true; do wget -q -O- URL; done

You will start seeking a bunch of OK! s as the load generator continuously hits the application.


deploymentconfig.apps.openshift.io/hpademo resource requirements updated
We have set the CPU request (initial allocation) as 200 millicores and limit (maximum allocation) to 500 millicores. So when we ask HPA to scale based on percentage workload, it measures based on these numbers.


curl -Lo hey https://storage.googleapis.com/hey-release/hey_linux_amd64
c) adding it in path . 
    /usr/local/bin/ path:
    $ chmod +x hey
    $ sudo mv hey /usr/local/bin/

   
   hey -c 50 -z 10s "http://ieopetclinic-ieopetclinic.apps.ocp43.itblab.uspto.gov/?sleep=3&upto=10000&memload=100"
e) oc get hpa -w

