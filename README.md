# Reddit-Clone-with-Kubernetes

## For Prerequisites [click here](https://trainwithshubham.hashnode.dev/prerequisite-for-deployment-of-a-reddit-copy-on-kubernetes-with-ingress-enabled)

## Step 1 Clone the source code

First step is to clone the source code you can do it using bellow command

```bash
git clone https://github.com/sanket363/Reddit-Clone-with-Kubernetes.git
```

## Step 2 Containerize the Application using Docker

Write a Dockerfile as below

```bash
FROM node:19-alpine3.15

WORKDIR /reddit-clone

COPY . /reddit-clone

RUN npm install 

EXPOSE 3000

CMD ["npm","run","dev"]
```

## Step 3 Building Docker-Image

Build a Docker Image

```bash
docker build -t <image-name> .
```

## Step 4 Push the image to DockerHub

For pushing the image to DockerHub you will need to login to the DockerHub account first for that use below command

```bash
docker login
```

You will be prompted to enter your docker account username and password.

If you don't have the DockerHub account then [click here](https://hub.docker.com/)

Once logged in to your DockerHub account
Use bellow command to push your build image to the DockerHub repository

```bash
docker tag <image-id> <docker-username>/<image-name:tag>

docker push <docker-username>/<image-name:tag>
```

## Step 5 Writing a kubernetes Manifest File

Now, You might be wondering what this manifest file in Kubernetes is. Don't worry, I'll tell you in brief.

When you are going to deploy to Kubernetes or create Kubernetes resources like a pod, replica-set, config map, secret, deployment, etc, you need to write a file called manifest that describes that object and its attributes either in yaml or json. Just like you do in the ansible playbook.

Of course, you can create those objects by using just the command line, but the recommended way is to write a file so that you can version control it and use it in a repeatable way.

- ## Writing a Deployment.yml file

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - name: reddit-clone
        image: snaket2628/reddit-clone
        ports:
        - containerPort: 3000
```

- ## Writing a Service.yml file

```bash
apiVersion: v1
kind: Service
metadata:
  name: reddit-clone-service
  labels:
    app: reddit-clone
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 31000
  selector:
    app: reddit-clone
```

## Step 7 Deploy the app to the kubernetes & Create a Service for it.

Now, we have a deployment file for our app and we have a running Kubernetes cluster, we can deploy the app to Kubernetes. To deploy the app you need to run following the command:

```bash
kubectl apply -f deployment.yaml -n <name-space>
```

Same for the service file

```bash
kubectl apply -f service.yaml -n <name-space>
```

## Configuring the Ingress

- Writing ingress.yml

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
spec:
  rules:
  - host: "domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
  - host: "*.domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
```

apply the ingress file with following command:

```bash
kubectl apply -f ingress.yml -n <name-space>
```

## Step 8 Expose the application

1. First We need to expose our deployment so use kubectl expose deployment reddit-clone-deployment --type=NodePort command.

2. You can test your deployment using curl -L http://{minikubeip}:31000. Port 31000 is defined in Service.yml

3. Then We have to expose our app service kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 &

### Test Ingress

Now It's time to test your ingress so use the curl -L domain/test command in the terminal.

## Final step 

Navigate to your chrome browser and enter http://{public-ip}:3000

🔥🔥🔥🔥 Boom your reddit clone is up and running 
