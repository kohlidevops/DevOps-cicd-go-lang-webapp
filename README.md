# DevOps-cicd-go-lang-webapp

**How to run go lang app locally?**

Install go lang on your system

For my case, it is ubuntu22 system

```
sudo apt-get update -y
sudo apt install golang-go
go version
```

**To clone the go lang webapp repo**

```
git clone https://github.com/kohlidevops/go-web-app.git
cd go-web-app
```

**To build the app first**

```
cd go-web-app
go build -o main .   //It will create build name called main in the root directory of the project
ls
```

**To run the go lang webapp**

```
cd go-web-app
./main   //which is created just now
```

**To access the go lang webapp through browser**

App is running with port 8080

```
http://15.207.222.118:8080/courses
```

<img width="954" alt="image" src="https://github.com/user-attachments/assets/947174ce-b3d7-4387-b460-7910a387208c">

**How to run the go lang app as docker container?**

We have to create a Dockerfile to build a docker image and run this image as a container. I have created multistage Dockerfile to avoid more image size and ensure the security - Just created a distroless image.

```
# Containerize the go application that we have created
# This is the Dockerfile that we will use to build the image
# and run the container
# Start with a base image
FROM golang:1.22.5 as base

# Set the working directory inside the container
WORKDIR /app

# Copy the go.mod and go.sum files to the working directory
COPY go.mod ./

# Download all the dependencies
RUN go mod download

# Copy the source code to the working directory
COPY . .

# Build the application
RUN go build -o main .

#######################################################
# Reduce the image size using multi-stage builds
# We will use a distroless image to run the application
FROM gcr.io/distroless/base

# Copy the binary from the previous stage
COPY --from=base /app/main .

# Copy the static files from the previous stage
COPY --from=base /app/static ./static

# Expose the port on which the application will run
EXPOSE 8080

# Command to run the application
CMD ["./main"]
```

You can copy the Dockerfile above or use below url to download

```
https://github.com/kohlidevops/go-web-app-devops/blob/main/Dockerfile
```

**Install docker and configure on your local machine**

```
sudo apt-get install docker.io -y
sudo chmod 777 /var/run/docker.sock
```

**To build the docker image**

```
cd go-web-app
//make ensure Dockerfile available in the project root directory
docker build -t latchudevops/go-web-app:v1 .
docker images
```

<img width="722" alt="image" src="https://github.com/user-attachments/assets/c393afbd-9c84-40c9-90db-d565a9f063d0">

**To run a docker image as container**

```
docker run -p <host-port>:<container-port> -it <username>/<app-name>:<version>
docker run -p 8080:8080 -it latchudevops/go-web-app:v1
```

<img width="716" alt="image" src="https://github.com/user-attachments/assets/d153faac-dfbc-4bf5-a994-08fc8f0b3132">

I can able to access the go lang webapp through my browser

<img width="958" alt="image" src="https://github.com/user-attachments/assets/d2e0be91-58a1-4a78-8780-49f32210a72f">

**To push the docker image to the Docker Hub**

First login to your Docker Hub and push this image to the registry

```
docker login
username - latchudevops
password - *******
```

<img width="917" alt="image" src="https://github.com/user-attachments/assets/6f9b8e4b-b440-496e-828f-ba83510c4092">

**To push the image**
```
docker push latchudevops/go-web-app:v1
```

<img width="836" alt="image" src="https://github.com/user-attachments/assets/448ca520-8778-4fed-81ae-63b9f2504673">

The image successfully pushed into the docker registry

<img width="859" alt="image" src="https://github.com/user-attachments/assets/30cf583b-feff-4498-bb2b-a1c63e7f94ea">

**To create a deployment, service and ingress manifests file**

To create a k8 folder and create a manifests folder inside the k8 folder - Then create a deployment yaml file.

**To create a deployment yaml file**

https://github.com/kohlidevops/go-web-app-devops/blob/main/k8s/manifests/deployment.yaml

```
# This is a sample deployment manifest file for a simple web application.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-web-app
  labels:
    app: go-web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-web-app
  template:
    metadata:
      labels:
        app: go-web-app
    spec:
      containers:
      - name: go-web-app
        image: latchudevops/go-web-app
        ports:
        - containerPort: 8080
```

**To create a service file**

https://github.com/kohlidevops/go-web-app-devops/blob/main/k8s/manifests/service.yaml

```
# Service for the application
apiVersion: v1
kind: Service
metadata:
  name: go-web-app
  labels:
    app: go-web-app
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: go-web-app
  type: ClusterIP
```

**To create a Ingress yaml file**

https://github.com/kohlidevops/go-web-app-devops/blob/main/k8s/manifests/ingress.yaml

```
# Ingress resource for the application
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: go-web-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: go-web-app.local
    http:
      paths: 
      - path: /
        pathType: Prefix
        backend:
          service:
            name: go-web-app
            port:
              number: 80
```

Now! we have to check all the yaml files are valid using kubernetes cluser. For that, we have to create a Amazon Elastic Kubernetes Cluster using EKSCTL

To create an EKS from your local machine. For my case it is ubuntu22 machine

Pre-req

1. Install a AWS CLI
2. Install kubectl cli
3. Install eksctl

Install a AWS CLI

You can use below link to install or use below snippet

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```
cd /home/ubuntu/
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

![image](https://github.com/user-attachments/assets/738c358c-20cc-473a-b67a-650cb86e6953)

Install a kubectl cli

You can use below link to install or use below snippet

https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.29.0/2024-01-04/bin/linux/amd64/kubectl
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.29.0/2024-01-04/bin/linux/amd64/kubectl.sha256
sha256sum -c kubectl.sha256
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
kubectl version --client
```

<img width="761" alt="image" src="https://github.com/user-attachments/assets/c9409a9c-40f5-4a88-ac65-6491d851b7d2">

Install a eksctl

You can use below link to install or use below snippet

https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up-eksctl.html

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

![image](https://github.com/user-attachments/assets/f06e6c2d-a72f-4712-a377-b38fd91a55c6)

**To configure AWS on your local machine**

To create a access / secret key and map to your system using aws configure command

```
aws configure
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [None]: 
Default output format [None]:
```

You can create an IAM role and associate with your EC2 instance - if you are assuming your local system as EC2 instance

**To Install a EKS using eksctl**

```
eksctl create cluster --name demo-cluster --region ap-south-1
```

My nodes are in available state

<img width="823" alt="image" src="https://github.com/user-attachments/assets/b7dfe8c8-6bd8-4edf-9e55-f6aec85cbce7">

```
kubectl get nodes
```

<img width="660" alt="image" src="https://github.com/user-attachments/assets/11aff24d-2cb5-4511-ab1d-b7dae6a3cd71">

To run the deployment yaml file from local system

```
cd /home/ubuntu/go-web-app
kubectl apply -f k8s/manifests/deployment.yaml
kubectl get pods
```

<img width="638" alt="image" src="https://github.com/user-attachments/assets/f5fe49f6-f641-4ebe-835a-9686198a944e">

To run the service yaml file from local system

```
cd /home/ubuntu/go-web-app
kubectl apply -f k8s/manifests/service.yaml
kubectl get svc
```

<img width="592" alt="image" src="https://github.com/user-attachments/assets/7d7e262c-7079-4b2f-a0c3-8ced30bae736">

To run the ingress yaml file from local system

```
cd /home/ubuntu/go-web-app
kubectl apply -f k8s/manifests/ingress.yaml
kubectl get ing
```

<img width="716" alt="image" src="https://github.com/user-attachments/assets/706534a3-613b-4858-80bf-0e3df4b966d9">

Still we can't able to access. Because address is not assigned yet to ingress means ingress controller not assigned any IP address.

As of now, All yaml files are created and its valid state only. We can check with NodePort for service. For that, we have edit the service and change the type as NodePort instad of ClusterIP and save it

```
kubectl edit svc go-web-app
type: NodePort
kubectl get svc
```

<img width="632" alt="image" src="https://github.com/user-attachments/assets/4212af62-40d9-453a-ba04-a3e37c8c3f3e">

Now you can check with node ip with node port 

http://65.2.150.163:32174/courses

<img width="943" alt="image" src="https://github.com/user-attachments/assets/cd972022-42bf-41ef-b153-01bb4c1ae71e">


To create a Nginx Ingress controller

To create a Nginx Ingress controller to create Network Loadbalancer on the AWS Cloud to expose your application through this ALB.

You can use below link to execute the ingress controller 

https://kubernetes.github.io/ingress-nginx/deploy/#aws

or you can use below snippet to run the nginx ingress controller

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```

What does Nginx Ingress controller will do? It just watch the Ingress that you created by yaml file then this ingress controller will create a loadbalancer for you.

<img width="928" alt="image" src="https://github.com/user-attachments/assets/5e2b844a-257d-41f6-b97b-12934aae4b61">

```
kubectl get pods -n ingress-nginx
kubectl get ing
```

<img width="914" alt="image" src="https://github.com/user-attachments/assets/62eb3852-c6c0-4cde-834b-25afca10c1c2">

**How Ingress controller mapped to the ingress?**

If you edit the Ingress controller, you can see the ingress-class=nginx

```
 --ingress-class=nginx
```

which is mapped ingress yaml file - You can edit the ingress yaml and check the content

```
spec:
  ingressClassName: nginx
  rules:
```

The Network Loadbalancer has been created as we can see below

<img width="754" alt="image" src="https://github.com/user-attachments/assets/01e29ed7-7fde-4448-aa55-e53a78b4404f">

**What will happen If i access the Loadbalancer URL?**

<img width="724" alt="image" src="https://github.com/user-attachments/assets/3e7ff7dd-b789-46f0-a5ab-bfa5599b0f85">

It wont access. Because I clearly mentioned in ingress yaml file that request should forward to the backend instance when host rule = go-web-app.local. Thats why we cant access the go lanfg app.

For that, we need to map this Loadbalancer IP to the given domain and we can check in the local system.

**To map to local system**

If you check the dnschecker as per below you can get 3 Ip address. You can copy any one of the IP and map with local system

<img width="761" alt="image" src="https://github.com/user-attachments/assets/fc8d2afb-70db-43c4-9d29-ac222e42399d">

```
For windows location path - C:\Windows\System32\drivers\etc\hosts
For Linux location path - /etc/hosts
```

<img width="499" alt="image" src="https://github.com/user-attachments/assets/4698fcb4-1c76-4db7-9555-882e3ccaf908">

I can ping from local systems

<img width="463" alt="image" src="https://github.com/user-attachments/assets/f0bbd65b-46f1-4699-8f11-30ef14067578">

Awesome! Now I can able to see the go lang webapp

<img width="946" alt="image" src="https://github.com/user-attachments/assets/59c89e99-4af3-4819-97fb-18a7855b9e74">

**To create a Helm chart to deploy the application**

**To install helm chart**

You can use below link to install or use commands

https://helm.sh/docs/intro/install/

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

<img width="866" alt="image" src="https://github.com/user-attachments/assets/eeacce29-f980-4174-b60e-8c0357f7d29b">

To create a helm chart

To create a new directory called helm inside the go-web-app

```
cd go-web-app
mkdir helm
cd helm
helm create go-web-app-chart
```

<img width="674" alt="image" src="https://github.com/user-attachments/assets/81721c77-96a7-4e5f-91f5-44b7979f7dc1">

If you list the go-web-app-chart, you can see the below files

<img width="590" alt="image" src="https://github.com/user-attachments/assets/3d47af70-6278-4902-b086-5214b959b0aa">

You can delete the charts directory and we can ignore this. So we have only 3 files such as Chart.yaml, templates folder, values.yaml

<img width="572" alt="image" src="https://github.com/user-attachments/assets/07e6af28-a187-443c-8b61-c8a8226aecaf">

If you list the templates folder inside go-web-app-chart

<img width="681" alt="image" src="https://github.com/user-attachments/assets/a965f2be-eaab-4ce4-ad36-47cc05ffb72b">

you can remove all the files from templates folder and copy all manifests file (deployment.yaml, service.yaml and ingress.yaml) to the templates folder.

<img width="554" alt="image" src="https://github.com/user-attachments/assets/3185a3c4-56ef-4abe-9aac-467c07d9f4c9">

Now all manifests files are available in the templates folder.

**To change the image name and tag syntax in deployment yaml**

To change the image name and tag in the deployment yaml file which is available in templates folder (as below)

path - /home/ubuntu/go-web-app/helm/go-web-app-chart/templates/deployment.yaml

```
# This is a sample deployment manifest file for a simple web application.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-web-app
  labels:
    app: go-web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-web-app
  template:
    metadata:
      labels:
        app: go-web-app
    spec:
      containers:
      - name: go-web-app
        image: latchudevops/go-web-app:{{ .Values.image.tag }}
        ports:
        - containerPort: 8080
````

To update the values yaml file in go-web-chart folder

To remove all the contents from values.yaml file

```
cd /home/ubuntu/go-web-app/helm/go-web-app-chart/
//remove all contents from values.yaml file
sudo nano values.yaml
//To add the below content
```

values.yaml file

use below code or link 

https://github.com/kohlidevops/go-web-app-devops/blob/main/helm/go-web-app-chart/values.yaml

```
# Default values for go-web-app-chart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: abhishekf5/go-web-app
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "v1"

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
```

To check whether helm chart is working

For this, we can remove all the things such as deployment, service and ingress using kubectl

```
kubectl delete deploy go-web-app
kubectl delete svc go-web-app
kubectl delete ing go-web-app
```

<img width="576" alt="image" src="https://github.com/user-attachments/assets/f0f48e4c-e32d-4557-995f-16049f8e4606">

Everything deleted! If you access the go lang web app then  its non found.

<img width="812" alt="image" src="https://github.com/user-attachments/assets/1fa6a496-9470-4da9-b34f-bd612e6329e0">

To deploy the web app through helm chart

```
cd /home/ubuntu/go-web-app/helm
helm install go-web-app ./go-web-app-chart/
```

The helm package has been installed

<img width="671" alt="image" src="https://github.com/user-attachments/assets/c35fe4b7-b9e7-4202-8fae-6cc400bf13f4">

You can check! that all yaml has been deployed

```
kubectl get deploy
kubectl get svc
kubectl get ing
```

If I access the same go-lang webapp page, I can able to acess

<img width="935" alt="image" src="https://github.com/user-attachments/assets/6cbeb4ac-a1ba-47a0-a90b-859ddfa0d8c1">

So, The helm chart is working as we expected. Now you can uninstall the helm package

```
cd /home/ubuntu/go-web-app/helm
helm uninstall go-web-app 
kubect get all
```



