Ref doc: https://www.ibm.com/docs/en/ibamoe/9.2.x?topic=installing-bamoe-maven-repository]

# BAMOE Local Setup Guide

## Prerequisites

- OCP cluster - 3 master and 3 worker
- Access to OpenShift CLI (`oc`)
- `scp` for copying files if needed
- Git (for version control) 

```bash
yum install git -y
```

## Install helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```


## Install Java (OpenJDK 11)

ref: https://docs.redhat.com/en/documentation/red_hat_build_of_openjdk/11/html-single/installing_and_using_red_hat_build_of_openjdk_11_on_rhel/index

```bash
yum install java-11-openjdk -y
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

## Deploy BAMOE Maven repo on OpenShift

Using quay.io/bamoe/maven-repository:9.2.0-ibm-0004

1. This case is simpler as you only need to set an environment variable with the image URL

```bash
export BAMOE_MAVEN_REPOSITORY_IMAGE=quay.io/bamoe/maven-repository:9.2.0-ibm-0004
```

2. Create the Deployment and Route

```bash
oc new-app ${BAMOE_MAVEN_REPOSITORY_IMAGE} --name bamoe-maven-repo
oc create route edge --service=bamoe-maven-repo
```

3. Get the Maven repository URL to be used in the next steps:

```bash
oc get route bamoe-maven-repo --output jsonpath={.spec.host}
```

## Deploy BAMOE Canvas on OpenShift with Helm Chart

### Installing the Chart

1. Use the following command to do a default installation:

```bash
helm install my-bamoe-canvas oci://quay.io/bamoe/canvas-helm-chart --version={VERSION}.
```

### OpenShift install

2. First, you may need to get the default OpenShift domain for your routes with the following command:

```bash
oc get ingresses.config cluster --output jsonpath={.spec.domain}
```

3. If you do not have access rights to this configuration, try creating a dummy Route resource and checking its domain.

```bash
helm pull oci://quay.io/bamoe/canvas-helm-chart --version={VERSION} --untar
helm install my-bamoe-canvas ./canvas-helm-chart --values ./canvas-helm-chart/values-openshift.yaml --set global.openshiftRouteDomain="<YOUR_OCP_ROUTE_DOMAIN>"
```

### Uninstalling the Chart

1. Follow the command to uninstall the my-bamoe-canvas deployment:

```bash
helm uninstall my-bamoe-canvas
```

## Installing BAMOE Management Console

### Deploying to Kubernetes or OpenShift with the Helm Chart
### Installing the Chart

1. Use the following command to do a default installation:

```bash
helm install my-bamoe-consoles oci://quay.io/bamoe/consoles-helm-chart --version={VERSION}
OpenShift install
```

2. First, you may need to get the default OpenShift domain for your routes with this command:

```bash
oc get ingresses.config cluster --output jsonpath={.spec.domain}
```

3. If you do not have access rights to this config, try creating a dummy Route resource and checking its domain.

Then, install the Helm Chart with the following command:

```bash
helm pull oci://quay.io/bamoe/consoles-helm-chart --version={VERSION} --untar
helm install my-bamoe-consoles ./consoles-helm-chart --values ./consoles-helm-chart/values-openshift.yaml --set global.openshiftRouteDomain="<YOUR_OCP_ROUTE_DOMAIN>"
```

### Uninstalling the Chart

1. To uninstall the my-bamoe-consoles deployment:

```bash
helm uninstall my-bamoe-consoles
```