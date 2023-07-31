# Code along
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

In the same directory, create a `requirements.txt` file:

```makefile
Flask==2.0.1
requests==2.26.0
```


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



In the same directory, create a `requirements.txt` file:

```makefile
Flask==2.0.1
```


## Step 3: Start Minikube

Start Minikube:

```bash
minikube start
```


## Step 4: Build the docker images

Set Docker environment to Minikube's:

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

You should now have ping-service and pong-service images in your Minikube Docker environment.

Create the deployments:

```bash
kubectl create deployment ping-service --image=ping-service:latest
kubectl create deployment pong-service --image=pong-service:latest
```



Update the deployments to use the local image:

```bash
kubectl patch deployment ping-service -p '{"spec":{"template":{"spec":{"containers":[{"name":"ping-service","imagePullPolicy":"Never"}]}}}}'
kubectl patch deployment pong-service -p '{"spec":{"template":{"spec":{"containers":[{"name":"pong-service","imagePullPolicy":"Never"}]}}}}'
```



### Exposing the deployments:

> If you have already exposed these deployments previously, you will get an error. If that's the case, you can delete the existing deployments and services first:

```bash
kubectl delete deployment ping-service
kubectl delete service ping-service

kubectl delete deployment pong-service
kubectl delete service pong-service
```


To expose the deployments run:

```bash
kubectl expose deployment ping-service --type=NodePort --port=5000
kubectl expose deployment pong-service --type=ClusterIP --port=5000
```



## Step 6: Test the services:

To get the URL of the ping-service, you can run:

```bash
minikube service ping-service --url
```
