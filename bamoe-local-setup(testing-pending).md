Refernece: 
https://www.ibm.com/docs/en/ibamoe/9.0.x?topic=started-initial-project-setup-walkthrough
BAMOE Canvas: https://www.ibm.com/docs/en/ibamoe/9.0.x?topic=installation-bamoe-canvas

# BAMOE Local Setup Guide

## Prerequisites

- OCP cluster - 3 master and 3 worker
- Access to OpenShift CLI (`oc`)
- `scp` for copying files if needed
- Git (for version control) 
```bash
yum install git -y
```

---

## Install Java (OpenJDK 11)

ref: https://docs.redhat.com/en/documentation/red_hat_build_of_openjdk/11/html-single/installing_and_using_red_hat_build_of_openjdk_11_on_rhel/index

```bash
yum install java-11-openjdk
java -version

apt update && apt install default-jdk -y
readlink -f $(which java)  # Note the JAVA path and copy the path till the before the /bin. now export the path till /bin in the next step/.
```

### Set JAVA_HOME and Update PATH

Add to `~/.bashrc`:

```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Verify Java Installation

```bash
echo $JAVA_HOME
java -version
```

Sample Output:

```

root@localoe1:~# echo $JAVA_HOME
/usr/lib/jvm/java-11-openjdk-amd64


root@localoe1:~# java -version
openjdk version "11.0.27" 2025-04-15
OpenJDK Runtime Environment (build 11.0.27+6-post-Ubuntu-0ubuntu122.04)
OpenJDK 64-Bit Server VM (build 11.0.27+6-post-Ubuntu-0ubuntu122.04, mixed mode, sharing)
```

---

## Install Apache Maven

1. Download from: https://maven.apache.org/install.html

   Example:
   ```bash
   wget https://dlcdn.apache.org/maven/maven-3/3.8.8/binaries/apache-maven-3.8.8-bin.tar.gz
   tar xzvf apache-maven-3.8.8-bin.tar.gz -C ~/
   ```

2. Add Maven to your PATH:
   ```bash
   echo 'export PATH=$HOME/apache-maven-3.8.8/bin:$PATH' >> ~/.bashrc
   source ~/.bashrc
   ```

3. Confirm Maven is working:
   ```bash
   mvn -v
   ```

Sample Output:
```
root@localoe1:~# mvn -v
Apache Maven 3.8.8 (4c87b05d9aedce574290d1acc98575ed5eb6cd39)
Maven home: /root/apache-maven-3.8.8
Java version: 11.0.27, vendor: Ubuntu, runtime: /usr/lib/jvm/java-11-openjdk-amd64
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "5.15.0-140-generic", arch: "amd64", family: "unix"
```

---

## Deploy BAMOE Canvas on OpenShift

BAMOE Canvas depends on 2 backend apps - extended services and Git CORS proxy.
Canvas → https://quay.io/repository/bamoe/canvas?tab=tags 
Extended Services → https://quay.io/repository/bamoe/extended-services?tab=tags 
Git CORS Proxy → https://quay.io/repository/bamoe/git-cors-proxy?tab=tags 

Note:
The exact same version needs to be used for a deployment with these three images to work.

Ensure pre-requisites are in place.

### Define variables that will be useful to name and label the resources created.

```bash
export APP_PART_OF=bamoe-canvas-app
export APP_NAME_EXTENDED_SERVICES=bamoe-extended-services
export APP_NAME_GIT_CORS_PROXY=bamoe-git-cors-proxy
export APP_NAME_BAMOE_CANVAS=bamoe-canvas
```

### Deploy Extended Services and label its resources

```bash
oc new-app quay.io/bamoe/extended-services:{VERSION} --name=$APP_NAME_EXTENDED_SERVICES       # Note version should be same across all 3 apps. Get the version from Quay.io under tags.
oc create route edge --service=$APP_NAME_EXTENDED_SERVICES
oc label services/$APP_NAME_EXTENDED_SERVICES app.kubernetes.io/part-of=$APP_PART_OF
oc label routes/$APP_NAME_EXTENDED_SERVICES app.kubernetes.io/part-of=$APP_PART_OF
oc label deployments/$APP_NAME_EXTENDED_SERVICES app.kubernetes.io/part-of=$APP_PART_OF
oc label deployments/$APP_NAME_EXTENDED_SERVICES app.openshift.io/runtime=golang
```

### Deploy Git CORS Proxy and label its resources

```bash
oc new-app quay.io/bamoe/git-cors-proxy:{VERSION} --name=$APP_NAME_GIT_CORS_PROXY           # Note version should be same across all 3 apps. Get the version from Quay.io under tags.
oc create route edge --service=$APP_NAME_GIT_CORS_PROXY
oc label services/$APP_NAME_GIT_CORS_PROXY app.kubernetes.io/part-of=$APP_PART_OF
oc label routes/$APP_NAME_GIT_CORS_PROXY app.kubernetes.io/part-of=$APP_PART_OF
oc label deployments/$APP_NAME_GIT_CORS_PROXY app.kubernetes.io/part-of=$APP_PART_OF
oc label deployments/$APP_NAME_GIT_CORS_PROXY app.openshift.io/runtime=nodejs
```

### Finally, deploy the BAMOE Canvas image, setting the environment variables required to connect it to the Extended Services and Git CORS Proxy backends.

```bash
oc new-app quay.io/bamoe/canvas:{VERSION} --name=$APP_NAME_BAMOE_CANVAS \                   # Note version should be same across all 3 apps. Get the version from Quay.io under tags.
  -e KIE_SANDBOX_EXTENDED_SERVICES_URL=https://$(oc get route $APP_NAME_EXTENDED_SERVICES --output jsonpath={.spec.host}) \
  -e KIE_SANDBOX_GIT_CORS_PROXY_URL=https://$(oc get route $APP_NAME_GIT_CORS_PROXY --output jsonpath={.spec.host})
oc create route edge --service=$APP_NAME_BAMOE_CANVAS
oc label services/$APP_NAME_BAMOE_CANVAS app.kubernetes.io/part-of=$APP_PART_OF
oc label routes/$APP_NAME_BAMOE_CANVAS app.kubernetes.io/part-of=$APP_PART_OF
oc label deployments/$APP_NAME_BAMOE_CANVAS app.kubernetes.io/part-of=$APP_PART_OF
oc label deployments/$APP_NAME_BAMOE_CANVAS app.openshift.io/runtime=js
```
### BAMOE Canvas instance should be up and accessible via the route created in this last step. 

To get it, run this command:

```bash
oc get route $APP_NAME_BAMOE_CANVAS --output jsonpath={.spec.host}
```
```bash
https://bamoe-canvas-default.apps.bamoe-local.cp.fyre.ibm.com
```
---

## To Deploy BAMOE via Docker, Docker Compose and or other installations
Refer: https://www.ibm.com/docs/en/ibamoe/9.0.x?topic=installation-bamoe-canvas

---

## Useful Links

- [BAMOE Canvas Docs](https://kiegroup.github.io/bamoe-docs/)
- [Kogito GitHub](https://github.com/kiegroup/kogito-runtimes)
- [Kogito] (https://kogito.kie.org/)
- [Drools] (https://www.drools.org/)
- [jBPM] (https://www.jbpm.org/)
- [Maven Official Site](https://maven.apache.org/)
- [OpenShift CLI](https://docs.openshift.com/)

---

## Maintained by

DivyaLakshmi (DevOps Engineer @ IBM)



important ref for bamoe install, upgrade and checks: https://www.ibm.com/docs/en/ibamoe/9.2.x?topic=started-initial-business-service-project-setup-walkthrough


install maven and java
setup maven repo
install helm

```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```