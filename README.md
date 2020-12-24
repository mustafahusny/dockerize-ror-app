# RoR application k8s deployment test

The application is created & dockerized following [this blog post](https://semaphoreci.com/community/tutorials/dockerizing-a-ruby-on-rails-application) with minor touches due to incorrect/out-dated versions.

## Development environment
Use docker-compose to build the local environment
```
$ export USER_ID=$(id -u)
$ export GROUP_ID=$(id -g)
$ docker-compose up
```

this will build the development image and mount the code from host to the container(s).

## Production environment with docker-compose
Use docker to build the production image then docker-compose to run the env with prod images.
```
export DOCKER_USERNAME=xxxx
$ docker build -t $DOCKER_USERNAME/drkiq -f Dockerfile.production .
$ docker build -t $DOCKER_USERNAME/dockerizing-ruby_nginx -f Dockerfile.nginx .
$ docker-compose -f docker-compose.prod.yml up
```


## Kuberentes setup
I'm using k3d instead of minikube, the differences are subtle in this example.

### Setting up k3d
We first need to setup a cluster to host our application. Use the following command to create a cluster and expose a loadbalancer on port 8081.
```
$ k3d cluster create ror -p 8081:80@loadbalancer
```

### Creating k8s objects
I've created k8s manifests from the production docker-compose using kompose and edited a few parts. A ConfigMap is used to pass the env variables to the application and traefik ingress is used instead of nginx (traefik is default in k3d).

Use the following command to create the objects.
```
$ cd .kuberentes; kubectl apply -f .
```

You can then check that everything is running with
```
$ kubectl get pods
```

Please note that these k8s manifests use pre-built images hosted on docker hub. If you inted to make changes, or use a different registry, make sure to push the images and change the manifests to use them.

### Testing
```
$ curl -H 'Host: drkiq' localhost:8081
```

You should see `The meaning of life is 42` wrapped in some HTML.

> many people _think_ that the meaning of life is 42, while actually 42 is the answer to the ultimate question about life, the universe and everything. The quesiton itself is unkown. DNA fan here <3

### Removing cluster
```
$ k3d cluster delete ror
```
