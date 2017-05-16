h1. How to configure a CI/CD pipeline for SBC-CFMS on OpenShift

- Create a project to house the Jenkins instance that will be responsible for promoting application images (via OpenShift ImageStreamTagS) across environment; the exact project name used was "sbc-cfms".
- Create the BuildConfiguration within this project using the ```oc``` command and "sbc-cfms-build-template.json" file in the templates directory:

```
oc process -f sbc-cfms-build-template.json -v NAME=<product-name> -v SOURCE_REPOSITORY_URL=<github url> -v SOURCE_REPOSITORY_REF=<branch or ref> | oc create -f -```
```

For example:

```
oc process -f sbc-cfms-build-template.json -v NAME=sample-name -v SOURCE_REPOSITORY_URL=https://github.com/samplerepo/location.git -v | oc create -f -```
```

- Deploy a Jenkins instance with persistent storage into the tools project using the web gui
- Install the Promoted Builds Jenkins plugin
- Configure a job that has an OpenShift ImageStream Watcher as its SCM source and promotion states for each environment
- In each promotion configuration, tag the target build's image to the appropriate promotion level; this was done using a shell command because the OpenShift plugins do not appear to handle parameter subsitution inside promotions properly.
- Create an OpenShift project for each "environment" (e.g. DEV, TEST, PROD, DEMO, TRAIN)
- Configure the access controls to allow the Jenkins instance to tag imagestreams in the environment projects, and to allow the environment projects to pull images from the esm project:
 
```
oc policy add-role-to-user system:image-puller system:serviceaccount <env-name>:default -n <target env>
oc policy add-role-to-user edit system:serviceaccount:<target env>:default -n <env-name>
```
 
- Use the JSON files in this directory  and `oc` tool to create the necessary resources within each project:

```
oc process -f esm-environment-template.json -v NAME=esm-<env-name>,APPLICATION_DOMAIN=esm-<env-name>.pathfinder.bcgov,APP_IMAGE_NAMESPACE=esm,APP_DEPLOYMENT_TAG=<env-name> | oc create -f -
```

For example:

```
```

h1. Environment

At this time there is only one environment, the Dev environment.


h1. How to access Jenkins

- Login to https://jenkins-sbc-cfms.pathfinder.gov.bc.ca with the username/password that was provided to you.

h1. How to access OpenShift for ESM

h2. Web UI
- Login to https://console.pathfinder.gov.bc.ca:8443; you'll be prompted for GitHub authorization.

h2. Command-line (```oc```) tools
- Download OpenShift [command line tools](https://github.com/openshift/origin/releases/download/v1.2.1/openshift-origin-client-tools-v1.2.1-5e723f6-mac.zip), unzip, and add ```oc``` to your PATH.  
- Copy command line login string from https://console.pathfinder.gov.bc.ca:8443/console/command-line.  It will look like ```oc login https://console.pathfinder.gov.bc.ca:8443 --token=xtyz123xtyz123xtyz123xtyz123```
- Paste the login string into a terminal session.  You are no authenticated against OpenShift and will be able to execute ```oc``` commands. ```oc -h``` provides a summary of available commands.

h1. Project contents

- The "servicebc-customer-flow-tools" project contains the Jenkins instance and the other servicebc-customer-flow-* projects contain different "environments".  The names are self-explanatory.

h1. Background reading/Resources

[Free OpenShift book](https://www.openshift.com/promotions/for-developers.html) from RedHat – good overview

[Red Hat Container Development Kit](http://developers.redhat.com/products/cdk/overview/)

OpenShift CI/CD pieline Demos:

- https://www.youtube.com/watch?v=65BnTLcDAJI
- https://www.youtube.com/watch?v=wSFyg6Etwx8


Transport endpoint is not connected healthcheck solution:

As soon as the volume 'goes away' your application will also be torn down 
and unavailable until such time as the glusterFS/NFS/Remote mount comes back.  
This is because the healthcheck once failed with kill the pod(s).  This may or 
may not be desirable based on your individual application behaviors.

Liveness snippet to put in your deployment config:
    livenessProbe:
    exec:
      command: [sh, /opt/app-root/src/scripts/mount_test.sh]
    initialDelaySeconds: 10
    timeoutSeconds: 5
    periodSeconds: 60
    successThreshold: 1
    failureThreshold: 3


Mount detection script:

https://github.com/bcgov/esm-server/blob/develop/scripts/mount_test.sh



Environment Variables:

MOUNT_POINT_CHECK=/remote/dir

MOUNT_POINT_TIMEOUT=seconds
  

   
