# Udagram_as_Microservices
 Udacity CloudDeveloper Nanodegree Project 3

Project Link: https://github.com/mohankrishna225/Udagram_as_Microservices

<img src ="Screenshots/Docker version2.png">

For deployment Version check git to version-2 and check the deployment files

# Udagram

Udagram is a simple cloud application developed alongside the Udacity Cloud Engineering Nanodegree. It allows users to register and log into a web client, post photos to the feed, and process photos using an image filtering microservice.

## Getting started

### Prerequisites
The following tools need to be installed on your machine:

- [Docker](https://www.docker.com/products/docker-desktop)
- [AWS CLI](https://aws.amazon.com/cli/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [KubeOne](https://github.com/kubermatic/kubeone)

Furthermore, you need to have:
- an [Amazon Web Services](https://console.aws.amazon.com) account
- a [DockerHub](https://hub.docker.com/) account

### Clone the repository

Clone the repository on your local machine:

```
git clone https://github.com/mohankrishna225/Udagram_as_Microservices.git
```

### Create an S3 bucket

The application uses an S3 bucket to store the images so an AWS S3 Bucket needs to be created

#### Permissions

Save the following policy in the Bucket policy editor:

```JSON
{
 "Version": "2012-10-17",
 "Id": "Policy1565786082197",
 "Statement": [
 {
 "Sid": "Stmt1565786073670",
 "Effect": "Allow",
 "Principal": {
 "AWS": "__YOUR_USER_ARN__"
 },
 "Action": [
 "s3:GetObject",
 "s3:PutObject"
 ],
 "Resource": "__YOUR_BUCKET_ARN__/*"
 }
 ]
}
```
Modify the variables `__YOUR_USER_ARN__` and `__YOUR_BUCKET_ARN__` by your own data.

#### CORS configuration

Save the following configuration in the CORS configuration Editor:

```XML
<?xml version="1.0" encoding="UTF-8"?>
 <CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
 <CORSRule>
 <AllowedOrigin>*</AllowedOrigin>
 <AllowedMethod>GET</AllowedMethod>
 <AllowedMethod>POST</AllowedMethod>
 <AllowedMethod>DELETE</AllowedMethod>
 <AllowedMethod>PUT</AllowedMethod>
 <MaxAgeSeconds>3000</MaxAgeSeconds>
 <AllowedHeader>Authorization</AllowedHeader>
 <AllowedHeader>Content-Type</AllowedHeader>
 </CORSRule>
</CORSConfiguration>
```

## Deploy on local

`Docker` is used to start the application on the local environment

The variables below need to be added to your environment:

```
POSTGRESS_USERNAME=udagram
POSTGRESS_PASSWORD=local
POSTGRESS_DB=udagram
POSTGRESS_HOST=db
WT_SECRET=mySecret
AWS_BUCKET=__YOUR_AWS_BUCKET_NAME__
AWS_REGION=__YOUR_AWS_BUCKET_REGION__
AWS_PROFILE=__YOUR_AWS_PROFILE__
```

Replace the variables `__YOUR_AWS_BUCKET_NAME__`, `__YOUR_AWS_BUCKET_REGION__` and `__YOUR_AWS_PROFILE__` by your own information

Build the images by running:

```
docker-compose -f docker-compose-build.yaml build --parallel
```

Start the application and services:

```
docker-compose up
```

The application is now running at http://localhost:8100

## Deploy on AWS

The application is running in a Kubernetes Cluster on AWS.



### Build the production images

At first, set these variables to your environment 

```
POSTGRESS_USERNAME=__YOUR_MASTER_USERNAME__
POSTGRESS_PASSWORD=__YOUR_MASTER_PASSWORD__
POSTGRESS_DB=__YOUR_INITIAL_DATABASE_NAME__
POSTGRESS_HOST=__YOUR_AMAZON_RDS_DB_HOST__
JWT_SECRET=__YOUR_JWT_SECRET__
AWS_BUCKET=__YOUR_AWS_BUCKET_NAME__
AWS_REGION=__YOUR_AWS_BUCKET_REGION__
AWS_PROFILE=__YOUR_AWS_PROFILE__
AWS_CREDENTIALS=`cat ~/.aws/credentials`
APP_URL=http://__YOUR_FRONTEND_SERVICE_URL__:8100
```

Replace the values by your data. `__YOUR_FRONTEND_SERVICE_URL__` can be retrieved using the command:

```
kubectl get svc
```

Add the reverseproxy URL to the file `frontend/src/environments/environment.prod.ts`

You can also retrieve the reverse proxy URL by running

```
kubectl get svc
```

Create a docker build file with the following content

```YAML
version: "3"
services:
 reverseproxy:
 build:
 context: .
 image: __YOUR_DOCKERHUB_NAME__/udagram-reverseproxy
 backend_user:
 build:
 context: ../../api-user
 image: __YOUR_DOCKERHUB_NAME__/udagram-api-user
 backend_feed:
 build:
 context: ../../api-feed 
 image: __YOUR_DOCKERHUB_NAME__/udagram-api-feed
 frontend:
 build:
 context: ../../frontend 
 args:
 - BUILD_ENV=production
 image: __YOUR_DOCKERHUB_NAME__/udagram-frontend
```

Replace ```__YOUR_DOCKERHUB_NAME__``` by your own DockerHub account.

Build the images by executing:

```
docker-compose -f __YOUR_DOCKER_BUILD_FILE__ build --parallel
```

Push your images to your Docker Hub

```
docker-compose -f __YOUR_DOCKER_BUILD_FILE__ push
```


etup Kubernetes environment
You will need to install the kubectl command. Open a new terminal within the project directory and run:

Generate encrypted values for aws credentials, Database User Name, and Database Password using bcrypt and put the values into aws-secret.yaml and env-secret.yaml files
Load secret files:
```
kubectl apply -f aws-secret.yaml
kubectl apply -f env-secret.yaml
```
Load config map: kubectl apply -f env-configmap.yaml
Apply Deployments:
```
kubectl apply -f backend-feed-deployment.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f backend-user-deployment.yaml
```
Apply Services:
```
kubectl apply -f backend-feed-service.yaml
kubectl apply -f backend-user-service.yaml
kubectl apply -f frontend-service.yaml
Deploy reverse proxy, has to be done after the services are running:
kubectl apply -f reverseproxy-deployment.yaml
kubectl apply -f reverseproxy-service.yaml
Perform port forwarding (each needs to be run in a separate terminal window and left running)
kubectl port-forward service/frontend 8100:8100
kubectl port-forward service/reverseproxy 8080:8080
```
Check Status:
```
kubectl get nodes
kubectl get pod --all-namespaces
kubectl get svc
kubectl get configmaps
kubectl get secrets
kubectl describe secret/env-secret
```




##Building a second version for deployment

Change to version: 2 in deployment configurations and apply them.
```
kubectl apply -f backend-feed-deployment.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f backend-user-deployment.yaml
kubectl apply -f reverseproxy-deployment.yaml
```

Now observe the changes in pods.
```
kubectl get pods
kubectl get rs
kubectl get deployments
```



