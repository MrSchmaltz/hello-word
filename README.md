### Tip: All files can be found in [GitHub](https://github.com/MrSchmaltz/hello-word)
## Task
Build a simple and EKS Docker Stack with Terraform to run a NodeJS App to say Hello World, fed by traffic from an Application Load Balancer.

## Technologies
- AWS EKS
- npm
- Terraform
- NodeJS
- Docker
- Kubernetes
- LoadBalance
- Linux Ubuntu
- Kubectl
- Docker hub

## Recipe
Login to AWS with AWS CLI and install all the above technologies and create yourself account where necessary.

## Terraform
Following all process from [Hashicorp](https://learn.hashicorp.com/tutorials/terraform/eks) ending on the deployments so do not need to do Deploy and access Kubernetes Dashboard.
    
### Configure kubectl
Run the following command to retrieve the access credentials for your cluster and automatically configure kubectl.
```
aws eks --region $(terraform output region) update-kubeconfig --name $(terraform output cluster_name)
```

## Kubernetes
[Install Kubectl]( https://kubernetes.io/docs/tasks/tools/install-kubectl/)
Find how to Install kubectl from this page.

[Expose External IP]( https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/)
Exposing an External IP Address to Access an Application in a Cluster process.
Following the process from the pages above with my customisation to point to my app on Docker Hub.

### 1 Creating a service for an application running in five pods
Create a fodler for your app.
```
mkdir app
cd app
```
Downlaod the laod balance file.
```
kubectl apply -f https://k8s.io/examples/service/load-balancer-example.yaml
```
Or create a file in your computer if you wish to customise as per my exemple below.
Custon changes to my script.
Change the line "- image: gcr.io/google-samples/node-hello:1.0" to point to my repository "- image: docker.io/vschmaltz/hello-world" on Docker Hub and load my Hello World app.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: load-balancer-example
  name: hello-world
spec:
  replicas: 5
  selector:
    matchLabels:
      app.kubernetes.io/name: load-balancer-example
  template:
    metadata:
      labels:
        app.kubernetes.io/name: load-balancer-example
    spec:
      containers:
      - image: docker.io/vschmaltz/hello-world
        name: hello-world
        ports:
        - containerPort: 8080
```
Run the yaml script by this below command:
```
kubectl apply -f load-balancer-example.yaml

deployment.apps/hello-world created
```

### 2 Check if it works with below command
``` 
kubectl get replicasets

NAME                    DESIRED   CURRENT   READY   AGE
hello-world-697d76678   5         5         5       2m42s
```

### 3 Create a Service object that exposes the deployment
``` 
kubectl expose deployment hello-world --type=LoadBalancer --name=my-service

service/my-service exposed
```

### 4 Display information about the Service
```
kubectl get services my-service

NAME         TYPE           CLUSTER-IP      EXTERNAL-IP                                                              PORT(S)          AGE
my-service   LoadBalancer   172.20.89.199   ad6c509a886234dcd91d5311f74cdbae-368390956.us-east-2.elb.amazonaws.com   8080:30596/TCP   57s
``` 
### 5 Check if you app is up and running
```
curl ad6c509a886234dcd91d5311f74cdbae-368390956.us-east-2.elb.amazonaws.com:8080

Hello World!
```
### 6 Display detailed information about the Service
```
kubectl describe services my-service

Name:                     my-service
Namespace:                default
Labels:                   app.kubernetes.io/name=load-balancer-example
Annotations:              <none>
Selector:                 app.kubernetes.io/name=load-balancer-example
Type:                     LoadBalancer
IP:                       172.20.89.199
LoadBalancer Ingress:     ad6c509a886234dcd91d5311f74cdbae-368390956.us-east-2.elb.amazonaws.com
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30596/TCP
Endpoints:                10.0.1.108:8080,10.0.1.125:8080,10.0.3.200:8080 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age    From                Message
  ----    ------                ----   ----                -------
  Normal  EnsuringLoadBalancer  4m23s  service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer   4m21s  service-controller  Ensured load balancer
```

### Folow with below commands if you wish delete your Kubernetes  LoadBalance and Replicas with commands.
### 1 To delete the LoadBalance Service, enter this command:
```
kubectl delete services my-service

service "my-service" deleted
```

### 2 To delete the Deployment, the ReplicaSet, and the Pods that are running the Hello World application, enter this command:
```
kubectl delete deployment hello-world
```

### Creating a Hello Word NodeJS app
[GitHub]( https://github.com/nodejs/docker-node/blob/master/README.md#how-to-use-this-image)
[Docker Node Container Example]( https://flaviocopes.com/docker-node-container-example/)

### 1 Create a directory for your app
```
mkdir node
cd node
```

### 2 Write your Hello World app.
[File on GitHub]( https://github.com/MrSchmaltz/hello-word/blob/master/app.js)
Copy the below code to file app.js
```
const express = require('express')
const app = express()

app.get('/', (req, res) => res.send('Hello World!'))
app.listen(8080, () => console.log('Server ready'))
```
Save and close nano.

### 3 Create your compose yaml file
[File on GitHub]( https://github.com/MrSchmaltz/hello-word/blob/master/docker-compose.yaml)
```
nano docker-compose.yaml
```
Copy the below code to your file docker-compose.yaml
```
version: "2"
services:
  node:
    image: "node:8"
    user: "node"
    working_dir: /home/node/app
    environment:
      - NODE_ENV=production
    volumes:
      - ./:/home/node/app
    ports:
     - 8080:8080
    command: "npm start"
```

### 4 Edit the pakage.json
[File on GitHub]( https://github.com/MrSchmaltz/hello-word/blob/master/pakage.json)
Add the below code line to file package.json
```
"start": "node app.js"
```
The final code should look like this below code:
```
{
  "name": "node",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node app.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1"
  },
  "description": ""
}
```
Save it and close nano.

### 5 Install npm
```
sudo apt install npm
```
And
```
npm init -y
```
```
npm install express
```

### 6 Run your docker
```
sudo docker-compose up -d
```
Use without -d if you wish to see the service running.

### 7 Access your app online
```
http:// Your IP : your port
```
Make sure the doors are open on the server as I have updated this one to listen on 8080 port on AWS EC2 in order to show hello word locally.

### 8 Creating an image file
[File on GitHub]( https://github.com/MrSchmaltz/hello-word/blob/master/Dockerfile)

### 9 Create a docker image by command.
```
docker build -t nodeapp .
```
Result
```
nodeapp_v2:latest
```

### 10 Now we can run a container from the image
Run container
```
docker run -d -p 8080:8080 --name node-app examplenode
 ```
Stop container
```
sudo docker stop node-app
```
 List container
```
docker container ls -l
```
 Remove container
```
docker container rm 978bcd06a78d
```
 List images
```
docker images
```

## Repository on docker hub
[Publishing Docker Image]( https://docs.github.com/en/free-pro-team@latest/actions/guides/publishing-docker-images)
[Docker Hub]( https://ropenscilabs.github.io/r-docker-tutorial/04-Dockerhub.html)

```
docker.io/username/hello-world
```

### Login
```
docker login --username=username
```

### List images
```
docker images
```

### Add tag
```
docker tag f4ccc42d64e9 username/hello-world
```

### push
```
docker push username/hello-world
```
Good luck and get in touch if you need so.
