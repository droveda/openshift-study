# Openshift for developers course

## Hosted OpenShift Dev Environment Sandbox
* Get a RedHat Account
  * redhat.com
* Provision a Developer Sandbox development cluster
  * developers.redhat.com/developer-sandbox
  * openshiftapps.com
* Open the OpenShift Console for the Sandbox
* Download oc
  * macos
    * go to the openshif dev/sanbox console, and search for the ? icon on the top, right
    * Select "Command Line tools"
    * Download the zip file for macos (Intel X86 or Apple ARM m1)
* Add oc to the PATH environment variable
  * put the oc executable file on a "tools" folder
  * export the oc, in order to execute it. For example:
    * vim ~/.zshrc
    * export PATH=$PATH:~/Tools
* Test the oc command:
  * go to the dev sandbox, on the top right select your username
  * "Copy login command" it will give the output command, something like this
    * oc login --token=<my-token> --server=<the-server-url>
* Util:
  * docker login quay.io
  * docker login docker.io (this is the default docker registry URL)
  * source test.env
    * echo $SOME_KEY

## Sign up for acccount if necessary
* hub.docker.com
* gitlab.com
* https://gitlab.com/practical-openshift/labs

## Utils docker
* docker build -t your-app:v01 .
* docker run -it -p 8081:8080 --name my-app your-app:v01
  * Base Dockerfile instructions
    * FROM (base image)
    * COPY (Copy our source file into the container)
    * ENV (define environmental variables)
    * RUN (run a command)
    * EXPOSE (documents that a port is available)
    * CMD (Command to run when container starts up)


## Openshift concepts
* oc status
* oc version
* Users
  * oc whoami
  * oc logout
  * oc login --token=<my-token> --server=<the-server-url>
* Projects
  * oc project
  * oc projects
  * oc new-project myproject
  * oc project <project name>
* pod
  * oc explain pod
  * oc explain pod.spec
  * oc explain pod.spec.containers
  * oc create -f pods/pod.yaml
  * oc get pods
  * oc rsh <pod name> (with this command you can start/connect a shell with a pod)
    * oc rsh hello-world-pod
  * oc delete pod <pod name>
  * oc get pods --watch
  * oc explain pod.spec.containers.env
  * oc port-forward pod/hello-world-pod 8080
  * oc describe pod hello-world-pod

## DeploymentConfigs
DeploymentConfigs is a collection of pods, this is new on openshif and not present in Kubernetes
* oc explain deploymentconfig
* Deploying an Image
  * oc new-app quay.io/practicalopenshift/hello-world --as-deployment-config
  * oc status
  * oc get svc
  * oc get dc
  * oc get istag
  * oc delete svc/hello-world
  * oc delete dc/hello-world
* Cleaning up using label selector
  * oc new-app quay.io/practicalopenshift/hello-world --as-deployment-config
  * oc describe dc/hello-world
    * the label selector is: app=hello-world
  * oc delete all -l app=hello-world
* Naming Deployments (DeploymentConfigs)
  * oc new-app quay.io/practicalopenshift/hello-world --name demo-app --as-deployment-config
  * oc describe dc/demo-app
  * oc delete all -l app=demo-app
* Deploying from Git
  * oc new-app <git repo URL>
  * Follow build progress
    * oc new-app https://gitlab.com/practical-openshift/hello-world.git --as-deployment-config
  * oc logs -f bc/hello-world
  * Check status and pods
    * oc status
    * oc get pods
  * oc delete all -l app=hello-world
* Replication Controllers
  * oc new-app quay.io/practicalopenshift/hello-world --as-deployment-config
  * oc get -o yaml dc hello-world
  * oc get rc
* Rollout and Rollback (DeploymentConfigs)
  * oc rollout latest dc/hello-world
  * oc get pods --watch
  * oc rollback dc/hello-world

## Services and Routes
* oc explain svc (this is the same as kubernetes services)
* oc explain svc.spec
* how to create a service:
  * oc create -f pods/pod.yaml
  * oc expose --port 8080 pod/hello-world-pod
  * oc status
  * oc create -f pods/pod2.yaml
  * testing the connection **pod2 calling pod**
    * oc rsh hello-world-pod-2
      * wget -qO- <service IP / port>
      * wget -qo- 172.30.13.208:8080
      * wget -qO- hello-world-pod:8080 (also works)
      * wget -qO- http://hello-world-pod:8080
      * typing **env** inside the pod, will print all the env variables (this is very cool)
* Exposing a Route (Routes expose an external DNS name that clients can use to access your service outside the cluster)
  * oc new-app quay.io/practicalopenshift/hello-world --as-deployment-config
  * oc expose svc/hello-world **this command will create a route for service hello-world**
  * oc status (to get the url)
  * curl http://hello-world-droveda-dev.apps.sandbox-m2.ll9k.p1.openshiftapps.com/
  * Routes are not parte of kubernetes, they are new to openshift
  * oc get -o yaml route/hello-world


## ConfigMaps
* oc explain configmap
* oc get cm
* oc get -o yaml cm/config-service-cabundle
* oc create configmap message-map --from-literal MESSAGE="Hello From ConfigMap"
* oc create -f pods/my-configmap.yaml
* oc create -f pods/pod-with-config-map.yaml

## Secrets
* oc explain secrets
* oc create secret generic message-secret --from-literal MESSAGE="Secret Message"
* oc get secret
* oc get -o yaml secret/message-secret
* oc set env dc/hello-world --from secret/message-secret
* oc create -f pods/secret-example.yaml
* oc create -f pods/pod-with-secret.yaml

## Images and Image Streams
* Images are a blueprint that openshift can use to create containers
* Types
  * ImageStream
  * ImageStreamTags
* oc new-app quay.io/practicalopenshift/hello-world --as-deployment-config
  * --> Creating resources ...
    imagestream.image.openshift.io "hello-world" created
    deploymentconfig.apps.openshift.io "hello-world" created
* oc get is
* oc get istag
* oc import-image --confirm quay.io/practicalopenshift/hello-world (this command will import the image to the Openshift ImageStream (It is similar to the dockerhub repo))
* oc new-app droveda-dev/hello-world (create a new deployment from a imported image)
* oc tag <original> <destination>
  * oc tag quay.io/image-name:tag image-name:tag
  * oc tag quay.io/practicalopenshift/hello-world:update-message hello-world:update-message 


### Lab Images (Private image)
* docker build -t droveda/images-labs:v01 .
* docker push droveda/images-labs:v01 (keep this one private in dockerhub)
* docker build -t droveda/images-labs:private-tag .
* docker push droveda/images-labs:private-tag (keep this one private in dockerhub)
* oc create secret docker-registry \
  lab-images-secret \
  --docker-server=$REGISTRY_HOST \
  --docker-username=$REGISTRY_USERNAME \
  --docker-password=$REGISTRY_PASSWORD \
  --docker-email=$REGISTRY_EMAIL
* oc secrets link default lab-images-secret --for=pull
* oc describe serviceaccount/default
* Now we should be able to create resources from a private docker-hub repository
  * oc import-image --confirm droveda/images-labs:private-tag
  * oc new-app droveda-dev/images-labs:private-tag
  * oc expose svc/images-labs


## Builds and BuildsConfigs
* build is simillar to the docker build command
* build config is how to build an imagem from the source code
* example
  * oc new-build https://gitlab.com/practical-openshift/hello-world.git
  * oc get -o yaml buildconfig/hello-world
  * oc get build
  * oc logs -f buildconfig/hello-world
  * oc start-build bc/hello-world
  * oc cancel-build bc/hello-world
* Webhooks in Action
* Building from a Branch
  * oc new-build https://gitlab.com/practical-openshift/hello-world.git#update-message
* How to start a Build in a Directory
  * oc new-build https://gitlab.com/practical-openshift/labs.git --context-dir hello-world
* Using Build Hooks
  1. Build Starts
  2. Build Runs
  3. Build Completes
  4. Post-build hook runs
  5. Image sent to ImageStream
1. oc set build-hook bc/hello-world --post-commit --script="Echo hello from build hook"
2. os describe bc/hello-world
3. oc start-build bc/hello-world
4. oc logs -f bc/hello-world
5. oc set build-hook bc/hello-world --post-commit --script="exit 1"
6. oc set build-hook bc/hello-world --post-commit --remove

## Source to Image (S2I)
It transforms your application source code into a container image.
* Flow of the build process:
  * Start -> Is there a docker file? Yes -> Use the Docker Stragegy, No -> Try Source Strategy


## Volumes
Filesystem mounted in Pods, Many Suppliers
- Config Maps, Secrets
- Attached Hard Disks
- Cloud Storage (S3, Google Compute Files)

### Volumes Labs (Empty Dir Volume)
* oc new-app quay.io/practicalopenshift/hello-world
* oc set volume deployment/hello-world \
  --add \
  --type emptyDir \
  --mount-path /empty-dir-demo
* oc get -o yaml deployment/hello-world
* oc get pods
  * oc rsh hello-world-7df78f575c-msdv9

### Volumes Mount a ConfigMap as a Volume
* oc create configmap cm-volume \
  --from-literal file.txt="ConfigMap file contents"
* oc set volume deployment/hello-world \
  --add \
  --configmap-name cm-volume \
  --mount-path /cm-directory
* oc get -o yaml deployment/hello-world
* oc get pods
  * oc rsh hello-world-7df78f575c-msdv9

### Volumes Mount a Secret as a Volume
* oc create -f labs/pods/secret-example.yaml
* oc set volume deployment/hello-world \
  --add \
  --secret-name lab-secret \
  --mount-path /secret-volume
* oc get -o yaml deployment/hello-world
* oc get pods
  * oc rsh hello-world-7df78f575c-msdv9

### Volumes, How to use other types of Volume Providers
* https://kubernetes.io/docs/concepts/storage/volumes/