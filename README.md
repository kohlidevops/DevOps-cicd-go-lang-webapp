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


