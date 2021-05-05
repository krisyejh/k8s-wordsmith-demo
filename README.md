# Kubernetes Wordsmith Demo

Wordsmith is the demo project shown at DockerCon EU 2017, where Docker announced that support for Kubernetes was coming to the Docker platform.

The demo app runs across three containers:

- [db](db/Dockerfile) - a Postgres database which stores words

- [words](words/Dockerfile) - a Java REST API which serves words read from the database

- [web](web/Dockerfile) - a Go web application which calls the API and builds words into sentences:

![The Wordsmith app running in Kubernetes on Docker for Mac](img/dockercon-barcelona-logo.svg)

## Build

The only requirement to build and run the app from source is Docker. Clone this repo and use Docker Compose to build all the images:

```
cd k8s-wordsmith-demo

docker-compose build
```

> Or you can pull pre-built images from Docker Hub using `docker-compose pull`.


## Deploy as a Docker Stack

The latest version of [Docker for Mac](https://www.docker.com/docker-mac) has Kubernetes built-in. 

Docker lets you use the simple [Docker Compose](https://docs.docker.com/compose/) file format to deploy complex applications to Kubernetes. You can deploy the wordsmith app to the local Kubernetes cluster using [docker-compose.yml](docker-compose.yml).

First use `docker version` to check whether Docker is running with Kubernetes or Docker Swarm as the orchestrator - Docker for Mac supports both orchestrators **at the same time**:

```
docker version -f '{{ .Client.Orchestrator }}'
```

> You can switch orchestrators with the `DOCKER_ORCHESTRATOR` environment variable, setting it to `kubernetes` or `swarm`.

Deploy the app to Kubernetes as a stack using the [compose file](docker-compose.yml):

```
export DOCKER_ORCHESTRATOR=kubernetes
docker stack deploy wordsmith -c docker-compose.yml
```

Docker for Mac includes the [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) command line, so you can work directly with the Kube cluster. Check the services are up, and you should see output like this:

```
$ kubectl get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
db           ClusterIP      None             <none>        55555/TCP        2m
kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP          38d
web          LoadBalancer   10.107.215.211   <pending>     8080:30220/TCP   2m
words        ClusterIP      None             <none>        55555/TCP        2m
```

Check the pods are running, and you should see one pod each for the database and web components, and five pods for the words API - which is specified as the replica count in the compose file:

```
$ kubectl get pods
NAME                   READY     STATUS    RESTARTS   AGE
db-8678676c79-h2d99    1/1       Running   0          1m
web-5d6bfbbd8b-6zbl8   1/1       Running   0          1m
words-858f6678-6c8kk   1/1       Running   0          1m
words-858f6678-7bqbv   1/1       Running   0          1m
words-858f6678-fjdws   1/1       Running   0          1m
words-858f6678-rrr8c   1/1       Running   0          1m
words-858f6678-x9zqh   1/1       Running   0          1m
```

Then browse to http://localhost:8080 to see the site. Each time you refresh the page, you'll see a different sentence generated by the API calls.


## Deploy Using a Kubernetes Manifest

You can deploy the same app to Kubernetes using the [Kubernetes manifest](kube-deployment.yml). That describes the same application in terms of Kubernetes deployments, services and pod specifications.

First remove the Kubernetes stack:

```
docker stack rm wordsmith
```

> Alternatively You can leave the Docker stack deployment running, and create a second deployment in a new Kubernetes namespace.

Now apply the manifest using `kubectl`:

```
kubectl apply -f kube-deployment.yml
```

Now browse to http://localhost:8081 and you will see the same site..
