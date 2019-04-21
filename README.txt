OC CLI Commands cheat-sheet
When using the oc command line tool, it is only possible to interact with one OpenShift cluster at a time. It is not possible even to open a separate shell on the same computer, as the same local user, and work on different OpenShift cluster at the same time.
This is because the state about the current login session is stored in the home directory of the local user running the oc command.

# oc got from; https://github.com/openshift/origin/releases/tag/v3.11.0

# a high level description of the main resource object types, and other concepts, can be obtained by running the command below
$ oc types

# to sign in, create a valid session
$ oc login https://cluster-context:portnumber
$ oc login https://cluster-context:portnumber --token=xyz
$ oc login https://cluster-context:portnumber --username myCleanUserName --password myCleanPassword
$ oc logout

# to switch between users, of course both with valid session;
$ oc login --username myUserName

# which user is being logging at the moment (also used for validating a valid session)
$ oc whoami
$ oc whoami --show-server
$ oc whoami -c

# checking for help commands
$ oc -h

# print the yaml(default) or json content with the configuration for the object "object-name" which of course, must be currently deployed
# when you use "$ oc get -o json" to query a resource object, it contains additional status information which can bind it to the current project and cluster. if you need to export a definition which can be used to create a new resource object in a different project or cluster, you should use "$ oc export" rather than "$ oc get"
$ oc export "object-name"
$ oc export "object-name" -o json

# the process command receives as parameter a template file which is then parsed and the place holders are changed given a resulting template ready to be created
$ oc process -f build-hello.json -p PARAM_1='value1' | oc create -f -
$ oc process -f build-hello.json -p PARAM_1='value1' | oc replace -f -
$ oc process -f build-hello.json -p PARAM_1='value1' | oc apply -f -

# create an object based on a template totally filled (no placeholder accepted at this point)
$ oc create -f mytemplate.json

# replace the definition of a resource object if it exists with that stored in a file. the format of the definition must be json or yaml. the metadata.name field of the definition must correspond to an existing resource object. if the resource object does not already exist, it will be created
$ oc apply -f definition.json
$ oc apply -f definition.yaml
$ oc apply -f ./dirWithDefinitionFiles


# help commands are also available on contexts, e.g; print information about the current active project
$ oc projects -h
$ oc projects
$ oc get projects

# create a new project
$ oc new-project "my-new-project-name" --description='My New Sample Project'

# switch to a project, referencing it remotly (it prints where the server is hosted on)
$ oc project sample-fabric-prj

# show a high level overview of the current project
$ oc status

# return all running pods, but even those failed ones are brought to assist with throubleshooting
$ oc get pods

# to create an interactive shell within the container running the database, we do it via '$ oc rsh' command
$ POD=`oc get pods --selector app=database -o custom-columns=name:.metadata.name --no-headers`
$ echo $POD
$ oc rsh $POD

# in order to perform a command on the remote running container
$ POD=`oc get pods --selector app=database -o custom-columns=name:.metadata.name --no-headers`
# executing kill command on the running container
$ oc rsh $POD kill -HUP 1

# return all objects dc=DeploymentConfigs, svc=Services, route=Routes
$ oc get dc,svc,route

# get a list of all the different resource object types
$ oc get

# query only for running Pods
$ oc get pods --selector app=myAppLabel -o jsonpath='{.items[?(@.status.phase=="Running")].metadata.name}'
$ pod() { local selector=$1; local query='?(@.status.phase=="Running")'; oc get pods --selector $selector -o jsonpath="{.items[$query].metadata.name}"; }
$ pod selectorLabelKey=selectorLabelValue

# it is also possible to show all existing objects available
$ oc get all
$ oc get all -o name
-- bc -> Build Config
-- is -> Image Stream; That image is the resulting built during build phase
-- dc -> Deployment Configurations;
-- svc -> Service (ClusterIP); Assigned a context Node internal IP for a Pod or a set of Pods. This IP is not available to the outside world
-- rc -> Replication Controller;
-- routes -> Routes; These are used for traffic comming from outside world and send then to internal Services, most of the time Services of type ClusterIP, so there's a close relation between services and services
-- po -> Pod; Holds all the containers

# create new application
# after this command got concluded, objects Service of type ClusterIP and DeploymentConfigs are created, so "oc status" might be useful to grab what has been changed
$ oc new-app docker-registry.svc.uk.paas.intranet.db.com/openshift3/hello-openshift

# if you have been given the name of an image to deploy and want to verify that it is valid from the command line, you can use the "$ oc new-app --search" command
$ oc new-app --search openshiftkatacoda/blog-django-py

# deploying with facility s2i for a python application hosted on a remote repository (github)
$ oc new-app python:latest~https://github.com/openshift-katacoda/blog-django-py --name blog

# creating a postgres database pod instance (with no storage set for persistent data)
$ oc new-app postgresql-ephemeral --name database --param DATABASE_SERVICE_NAME=database --param POSTGRESQL_DATABASE=sampledb --param POSGRESQL_USER=username --param POSGRESQL_PASSWORD=password

# similar to the new "$ oc new-app" command but a bit simpler, the "$ oc run" command below runs an image avoiding create aditional objects, such as service:ClusterIp. rather, the "$ oc run" command creates only the "Deployment config" alongside a pod for getting the contaienr image to be running
# run a dummy application pod which can be used to mount persistent volumes to facilitate copying files to a persistent volume
$ oc run dummy --image centos/httpd-24-centos7

# creating a Route object for the Service ClusterIP "hello-openshift"
$ oc expose svc hello-openshift

# removing the route "hello-openshift", but only it, the Service ClusterIP "hello-openshift" still alive
$ oc delete routes hello-openshift

# exporting the current configuration for the Service ClusterIP "hello-openshift"
$ oc export svc hello-openshift > hello-openshift-config.yaml

# exportnin configuration for json content
$ oc export svc hello-openshift -o json

# once we have all kubernetes files available, we can issue commands like;
$ oc create -f hello-openshift-config.yaml

# to add a label web to the service object for an application, and give it the value true;
$ oc label service/<serviceName> web=true

# to remove a label from a resource object, for example a service;
$ oc label service/<serviceName> web-

# dealing with labels
$ oc label --overwrite service hello-openshift app=foobar
$ oc label --overwrite route hello-openshift app=foobar
$ oc get all --selector=app=hello-openshift
$ oc get all --selector=app=app=foobar
$ oc get all --selector="app in (hello-openshift, foobar)"

# when you deploy an existing Docker-formatted container image which resides on an external image registry, the image will be pulled down and stored within the internal OpenShift image registry. The image will then be copied to any node in the OpenShift cluster where an instance of the application is run.
# in order to track the image that is pulled down, an Image Stream resource will be created. You can list what image stream resources have been created within a project by running the command;
$ oc get imagestream -o name
$ oc get ic -o name

# to see a description of the purpose of specific fields in the raw object, display documentation for objects, e.g;
$ oc explain Service
$ oc explain svc
$ oc explain Route
$ oc explain Pod
$ oc explain po
$ oc explain DeploymentConfig
$ oc explain dc

# create secrets based on "ssh private key"
$ oc secrets new my-secret-name ssh-privatekey=my-privatekey-rsa.pk

# show available secrets (only name)
$ oc get secrets

# remove all objects type pvc (persistent volume claim)
$ oc delete pvc --all

# always use oc describe to verify what labels have been applied and use "$ oc get all --selector" to verify what resources are matched before deleting any resources.
$ oc delete all --selector selectorkey=selectorvalue
$ oc delete all --selector="app in (hello-openshift, foobar)"
$ oc delete all --all

# creating a new build, based on a docker file simple content
$ oc new-build --name "build-name" -D $'FROM docker-registry.svc.uk.paas.intranet.db.com/openshift3/hello-openshift\nVOLUME ["/config"]' -o json | oc create -f -

# run a build parameterized as it is
$ oc start-build "build-name" --follow
$ oc start-build "build-name" --wait

# trigger a new build and deployment of the application where source code is taken from the current working directory of the local machine where the oc command is being run
$ oc start-build "build-name" --from-dir=.

# run a build aiming to have a resulting Image Stream (is) object based on a Dockerfile on the local host/machine
$ oc start-build "build-name" --from-file:./Dockerfile --follow

# for cancelling a running build
$ oc cancel-build "build-name"

# run a cluster locally (it is mandatory to have pre-installed docker up and running on the hosting machine)
$ oc cluster up
$ oc cluster down

# checking which is the current cluster
$ oc whoami --show-context

# get a list of all openshift ever logged into
$ oc config get-clusters

# list what users on the clusters I have logged in as, and which projects I have worked on
$ oc config get-contexts

# checking logs for a build config object
$ oc logs bc/myBuildConfig --follow

# to edit details of objects (default content edition is yaml)
$ oc edit <type>/<object name>
$ oc edit <type>/<object name> -o yaml
$ oc edit <type>/<object name> -o json

# monitoring progress of deployments
# the command will exit once the deployment has completed and the web application is ready
$ oc rollout status <type>/<object name>
$ oc rollout status dc/appName

# when a web application is made visible outside of the OpenShift cluster a Route is created. This enables a user to use a URL to access the web application from a web browser.
# a route is only usually used for web applications which use the HTTP protocol.
# a route cannot be used to expose a database for example, as they would typically use their own distinct protocol and routes would not be able to work with the database protocol.
# to setup port forwarding between a local machine and the database running on OpenShift you use the oc port-forward command
# remember the '$ oc port-forward' process still running on background, it can be checked either via '$ ps aux' and '$ jobs'
$ oc port-forward <pod-name> 
$ oc port-forward <pod-name> :<remote-pod-port>
$ oc port-forward <pod-name> <local-port>:<remote-pod-port>

# synchronize/copy files from/to remote running container
# copying file from remote container to local machine, on the current directory
$ oc rsync <pod-full-name>:/remote/pod/directory/file.txt .
# copying remote directory to the current machine on local directory
$ oc rsync <pod-full-name>:/remote/pod/directory .
# copying the contents of remote directory to the local "uploads" directory
$ oc rsync <pod-full-name>:/remote/pod/directory/. ./uploads
# copying a single file from local machine to the remote running container
$ oc rsync . <pod-full-name>:/remote/pod/directory --exclude=* --include=fileToBeUploaded.txt --no-perms
# for synchronization, where the oc copies from/to remote automatically, the "--watch" can be provided, this way a running process on background will keep live until be killed applying this synchronization
$ oc rsync currentLocalDirectory/. <pod-full-name>:/remote/pod/directory --no-perms --watch &
# specifying a strategy for copying resources from local host to the remote running container
$ oc rsync currentLocalDirectory/. <pod-full-name>:/remote/pod/directory --stratagy=tar
# copy content from remote to the local host, with directories/files in the local directory which are not found in the pod being deleted
$ oc rsync <pod-full-name>:/remote/pod/directory currentLocalDirectory/. --delete

# creating/claim a new volume and mounting it to a deployment config
$ oc set volume dc/myDeploymentConfigName --add --name=deploymentConfigMountName --claim-name=pvcStorageName --type pvc --claim-size=1G --mount-path /mnt/podMountDirectory

# unmounting the volume from a running container
$ oc set volume dc/myDeploymentConfigName --remove --name=deploymentConfigMountName

# mounting an already created volume pvc
$ oc set volume dc/myDeploymentConfigName --add --name=deploymentConfigMountName --claim-name=pvcStorageName --mount-path /mnt

# odo samples; https://github.com/openshift/odo
$ odo login -u myUserName -p myCleanPass
$ odo project create myProjectName
$ odo app create myApplicationName
$ odo app list
$ odo catalog list components
$ odo log -f

# odo creating and exposing components
$ odo create java backend --binary target/myapp-1.0.jar
$ odo push
$ odo create nodejs frontend
$ odo push
$ odo link backend --component frontend --port 8080
$ odo url create frontend
$ odo url list
