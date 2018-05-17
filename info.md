
requirements
*   text editor
*   kubectl
*   helm
*   git
*   vagrant
*   oracle virtualbox


Download and install Oracle VirtualBox and Vagrant
10 min

Download and install kubectl and helm
10 min

Register AWS account
~5 min

Register IAM account in the AWS account
~5 min



Register and download kublr box
3 min

10:43

Start Kublr box
~15 min - first time
~5 min - second time

Register IAM account secrets in Kublr box
~2 min

10:56

10:57

Create a new full Kublr platform
pwd: Password1!

Issues
1. it is not clear from the docs what size cluster should be ok (number of nodes and their size)
2. it is not clear from the docs whether letsencrypt in necessary
3. it is not clear from docs or UI whether logging and monitoring are centralized or local
4. It seems that logging and monitoring are not installed at all - neither centralized, nor individual
5. It seems that box still resizes disk


11:03
wait for platform to be deployed
11:23 - control plane installed



Create EFS accessible from nodes:
-   select vpc of the cluster
-   select subnets of the cluster nodes (cannot be modified later)
-   select default VPC security groups for zones (can be modified later)
~2 min
~3 min creation
testing on instance:
```
mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 MOUNT_TARGET_IP:/ efs
```


Modify file `build/efs-pv.yaml` putting correct endpoint in the spec.




```
# create namespace "build"
kc apply -f build/build-namespace.yaml

# create PV and PVC
kc apply -f build/efs-pv.yaml
kc apply -f build/efs-pvc.yaml

# testing or working with the PV
kc apply -f build/efs-pod.yaml
kc exec -n build -i -t efs -- sh
kc delete -f build/efs-pod.yaml

# create/start Jenkins
kc apply -f build/jenkins/jenkins-deployment.yaml
kc apply -f build/jenkins/jenkins-service.yaml

# Test Jenkins via port forwarding
# kc port-forward $(kc get pods -l app=jenkins -o 'jsonpath={.items[*].metadata.name}') 8080:8080

# create/start Nexus
kc apply -f build/nexus/nexus-deployment.yaml
kc apply -f build/nexus/nexus-service.yaml

# Test Nexus via port forwarding
# kc port-forward $(kc get pods -l app=nexus -o 'jsonpath={.items[*].metadata.name}') 8081:8081

# default Nexus login admin/admin123
```

In Jenkins:

1.  Install plugins

    Manage Jenkins > Manage Plugins > Available

    1.  Kubernetes
    2.  Pipeline
    3.  Timestamper
    4.  Git

    Mark "Restart"

2.  Manage Jenkins > Configure System > Add new cloud > Kubernetes
    1.  Kubernetes URL: https://kubernetes.default.svc.cluster.local
    2.  Kubernetes Namespace: `default`
    3.  Jenkins URL: http://jenkins.build.svc.cluster.local:8080

3.  Disable Job execution on the master:
    1.  Build Executor Status > Master: set # of executors to 0

4.  Create credentials to access nexus
    1.  Credentials > System > Global credentials (unrestricted) > Add Credentials

        -   ID: docker-registry
        -   Kind: Username with password
        -   Scope: Global
        -   Username: admin
        -   Password: admin123

    2.  Credentials > System > Global credentials (unrestricted) > Add Credentials
        -   ID: k8s-config
        -   Kind: Secret file
        -   upload config.yaml for the cluster

In Nexus:

1.  login admin/admin123
2.  create hosted docker repository:
    1.  name: docker0
    2.  HTTP port: 5000

Configure default namespace to work with private repository:

```
kubectl create secret docker-registry nexus-registry \
    --docker-server=nexus.build.svc.cluster.local:5000 \
    --docker-username=admin \
    --docker-password=admin123 \
    --docker-email=email@example.com

kubectl patch serviceaccount default \
    -p '{"imagePullSecrets": [{"name": "nexus-registry"}]}'
```


1.  Test job:

    -   Create a new job `test-pipeline` of type `Pipeline`.
    -   Use script from `build/jenkins/jobs/test-pipeline.Jenkinsfile`
    -   Run the job to verify that it works.

2.  Build Job:

    -   Create a new job `build-demo-app` of type `Multibranch Pipeline`.
    -   Branch Sources > Add source > Git
        -   Project Repository: https://github.com/kublr/workshop-build-pipeline-demo-app.git
        -   Behaviors > Filter by name (with wildcards)
            -   Include: `feature/*`
        -   Behaviors > Check out to matching local branch

# Istio

```
wget https://github.com/istio/istio/releases/download/0.7.1/istio-0.7.1-linux.tar.gz
tar xvf istio-0.7.1-linux.tar.gz
```

Install

```
cd istio-0.7.1

export PATH=$PWD/bin:$PATH

# run twice to create custom resources
kubectl apply -f install/kubernetes/istio.yaml
kubectl apply -f install/kubernetes/istio.yaml
```

# What is missing

Omitted for brevity:

-   security
    -   users, authentication, roles in Jenkins, Nexus, k8s etc
    -   secure credentials management in Jenkins
    -   limiting application images deployable to Kubernetes clusters

-   separate environment for build components, build jobs, app runtime dev/qa/prod

-   multi-cluster operations

-   helm packages for build pipelines components: jenkins, nexus, etc

-   Different AWS EFS shares for nexus and jenkins
-   cloud native storage (e.g. rook)

-   container images verification as a part of build process

-   microservices and gradual deployment
-   Continous delivery

    microservices platform like istio with gradual deployment scenarios

-   nexus images maintenance, e.g. cleanup old feature branch images

    nexus scripts/tasks, jenkins task via nexus API etc

-   kubernetes namespaces maintenance, e.g. cleanup of old namespaces

    scripts/tasks, jenkins tasks according to cleanup policy etc

-   Components maintenance: upgrading nexus, jenkins, kubernetes etc

-   Backup and DR - build and app data backup, recovery, replication

-   High availability
    -   k8s multi-master
    -   HA nexus, Jenkins
    -   Databases and FS

```
helm upgrade -i demo ./kubernetes/demo-app
```
