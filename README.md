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



