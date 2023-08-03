# Code along - Kubernetes: Ping Pong

## Before you begin

This tutorial assumes that you have already set up `minikube` and `kubectl`.
If not, see [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/) and [minikube](https://minikube.sigs.k8s.io/docs/start/) install guides.

> **Disclaimer**: As we move forward with this code-along, please note that we'll be working with Docker inside a Minikube environment. It's crucial to ensure your Docker CLI is correctly configured to interact with the Docker daemon inside your Minikube VM, rather than any other Docker environment on your machine. 
>
> To ensure this, we will need to execute the following command in the terminal:
>
>```bash
>eval $(minikube docker-env)
>```
>
> This change applies only to your current shell session. If you open a new terminal or a new shell session, you'll need to run this command again.


## Step 1: Create the 'Ping' service

First, create a new directory for your project and navigate into it:

```bash
mkdir pingpong-k8s && cd pingpong-k8s
```



Inside this directory, create a new directory called 'ping' and navigate into it:

```bash
mkdir ping && cd ping
```



Create a new file called `app.py` and put the following code in it:

```python
from flask import Flask
import requests
app = Flask(__name__)

@app.route('/')
def ping():
    try:
        response = requests.get('http://pong-service:5000')
        return "Received: " + response.text
    except Exception as e:
        return str(e)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```



This application will make a GET request to the 'pong' service at the '/' endpoint when it receives a GET request at the '/' endpoint.

To run this Python code, we need to install the necessary dependencies, in the same directory, create a `requirements.txt` file:

```makefile
Flask==2.0.1
requests==2.26.0
```

Now create a `Dockerfile` in the same directory:

```Dockerfile
FROM python:3.8-slim-buster

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

CMD [ "python3", "app.py" ]
```



This will create a Docker image with your Python application.


## Step 2: Create the 'Pong' service

Navigate back to your project directory and create a new directory called 'pong', then navigate into it:

```bash
cd ../ && mkdir pong && cd pong
```



Create a new file called `app.py` and put the following code in it:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def pong():
    return "Pong!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```



In the same directory, create a `requirements.txt` file:

```makefile
Flask==2.0.1
```



This application will return 'Pong!' when it receives a GET request at the '/' endpoint.

Now create a `Dockerfile` in the same directory:

```Dockerfile
FROM python:3.8-slim-buster

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

CMD [ "python3", "app.py" ]
```



## Step 3: Start Minikube

Start Minikube:

```bash
minikube start
```


## Step 4: Build the docker images

If you did not set the Docker environment to Minikube's before, run to do so now:

```bash
eval $(minikube docker-env)
```



Build the Docker images for ping-service and pong-service:

Inside the ping directory:

```bash
cd ping
docker build -t ping-service .
```



Inside the pong directory:

```bash
cd ../pong
docker build -t pong-service .
```


## Step 5: Deploy the applications to Minikube

At this point, we should have two Docker images, `ping-service` and `pong-service`, built and ready in our Minikube's Docker environment.

The next task is to create Kubernetes deployments for our services. A Deployment is a Kubernetes resource where we specify the desired state for our application. It allows Kubernetes to change the actual state to the desired state at a controlled rate.

### Creating the deployments:

```bash
kubectl create deployment ping-service --image=ping-service:latest
kubectl create deployment pong-service --image=pong-service:latest
```

Now, Kubernetes knows about our applications and how to run them, but it's still pulling the images from the default Docker environment. We need to tell Kubernetes to use the images we built inside the Minikube environment.


### Update the deployments to use local images:

We can update the deployments to use the local Docker images by patching them with an `imagePullPolicy` of `Never`. This means that Kubernetes will never try to pull the image from a registry, but instead, it will use the local Docker image.

```bash
kubectl patch deployment ping-service -p '{"spec":{"template":{"spec":{"containers":[{"name":"ping-service","imagePullPolicy":"Never"}]}}}}'
kubectl patch deployment pong-service -p '{"spec":{"template":{"spec":{"containers":[{"name":"pong-service","imagePullPolicy":"Never"}]}}}}'
```

> **Note:** Instead of patching the deployment through the cli, deployments are usually declared through YAML files. This is what the YAML file of the ping service would look like after patching the `imagePullPolicy`:
>
>```yaml
>apiVersion: apps/v1
>kind: Deployment
>metadata:
>  name: ping-service
>spec:
>  replicas: 1
>  selector:
>    matchLabels:
>      app: ping-service
>  template:
>    metadata:
>      labels:
>        app: ping-service
>    spec:
>      containers:
>      - name: ping-service
>        image: ping-service:latest
>        imagePullPolicy: Never
>```
>
>



### Exposing the deployments:

> **Note:** If you have already exposed these deployments previously, you might get an error. If that's the case, you can delete the existing deployments and services first:
>
> ```bash
> kubectl delete deployment ping-service
> kubectl delete service ping-service
>
> kubectl delete deployment pong-service
> kubectl delete service pong-service
>```


Run the following commands to expose the `ping-service` and `pong-service` deployments:

```bash
kubectl expose deployment ping-service --type=NodePort --port=5000
kubectl expose deployment pong-service --type=ClusterIP --port=5000
```

By running these commands, we create Kubernetes services that expose our deployments to network traffic. The `ping-service` is exposed externally through a static port on each Node in the cluster. On the other hand, the `pong-service` is exposed internally within the cluster. The `--port=5000` argument in both commands sets the network port through which the services will be accessible.


## Step 6: Test the services:

To get the URL of the ping-service, you can run:

```bash
minikube service ping-service --url
```

Put this url in your browser or use `curl` to test your PingPong kubernetes application.

Congratulations, you have successfully created a PingPong application in Kubernetes!

### Useful links

[`kubectl` cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

[Minikube documentation](https://minikube.sigs.k8s.io/docs/)
