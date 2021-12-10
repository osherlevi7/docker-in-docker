# docker-in-docker.
>Step by step process of setting up Docker image build in Kubernetes pod using Kaniko image builder.


<br />
<br />

### Kainko. 

* Kaniko -  is an open-source container image-building tool created by Google.
* There is a dedicated Kaniko executer image which builds the contianer images.
    $ gcr.io/kaniko-project/executor
<br />
this image containes only staic go binary and logic to push/pull images from/to registry.

* Kaniko accepts three arguments. A Dockerfile, build context and a remote registry to push the build image.
* When you deploy kaniko image it reads the Dockerfile and extracts the base image file system using the **FROM** instruction.
* The it executes each instruction from the Dockerfile and takes a snapshot in the userspace.
* After each snapshot, kaniko appends only the changed image layers to the base image and updates the image metadata. It happens for all the instructions in the Dockerfile.
* Finally, it pushes the image to the given registry.

<br />

**As you can see, all the image-building operations happen inside the Kaniko container’s userspace and it does not require any privileged access to the host.**
Kaniko supports the following type of build context.

-  GCS Bucket
-  S3 Bucket
- Azure Blob Storage
- Local Directory
- Local Tar
- Standard Input
- Git Repository

### On this project we'll use the **GitHub Repository**.


<br />

# Building Docker Image With, Kaniko, Github, Docker Registry & Kubernetes. 

- A valid Github repo with a Dockerfile: kaniko will use the repository URL path as the Dockerfile context
- A valid docker hub account: For kaniko pod to autheticate and push the built Docker image.
- Access to Kubernetes cluster: To deploy kaniko pod and create docker registry secret.


### start minikube. 
    $ minikube start 

### Genarate new docker-registry secret (DockerHub)
    $ kubectl create secret docker-registry dockercred \
    --docker-server=https://index.docker.io/v1/ \
    --docker-username=<dockerhub-username> \
    --docker-password=<dockerhub-password>\
    --docker-email=<dockerhub-email>

### Configure the pod.yml with the context & destination.
    $ args:
    - "--context=git://github.com/<user-name>/<repo>"
    - "--destination=<dockerhub-username>/<image-mane>:1.0"
    
### Deploy the pod.
    $ kubectl apply -f pod.yaml


**To validate the docker image build and push, check the pod logs.**
    $ kubectl logs kaniko --follow

