# Example Voting App

A simple distributed application running across multiple Docker containers.

yaml file (work on the Kubernetes cluster) : https://github.com/rineshsps/example-voting-app-kubernetes 

article: https://medium.com/@vikisquarez/docker-beginner-example-voting-application-207162338e79

video: https://accionlabs.udemy.com/course/learn-kubernetes/learn/lecture/21126242#notes 

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:5000](http://localhost:5000), and the `results` will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

Get pod  

```shell
kubectl get pods 
```

Get service  

```shell
kubectl get services 
```


To get inside a Redis database running  

```shell
kubectl exec -it <pod-name> -- /bin/sh

redis-cli

scan 0 
GET votes
type votes
LRANGE votes 0 10  

```

To get inside a Redis database running  

```shell
kubectl exec -it <pod-name> -- /bin/bash

psql -d <database> -U <username> -W

select datname  from pg_database;


---
psql -d <database> -U <username>

SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';


```



The `vote` web app is then available on port 31000 on each host of the cluster, the `result` web app is available on port 31001.

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## deploy in aks azure 

Reference : https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-portal?tabs=azure-cli 

#1 create Kubernetes service in aks 
#2 open shell cmd, then login with below cmd 

```shell
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

#3 clone the repo
```shell
git clone https://github.com/rineshsps/example-voting-app.git
```

#4 change the diectory 
```shell
cd example-voting-app/
```

#5 create 
```shell
kubectl create -f k8s-specifications/
```



## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
