# Assignment 3 - Kubernetes with Minikube


## Prerequisites: 
- VirtualBox
- ~20 GiB Disk Space

## Docs:
- https://github.com/kubernetes/minikube/blob/master/README.md
- https://kubernetes.io/docs/getting-started-guides/minikube/
- https://kubernetes.io/docs/concepts/
- https://kubernetes.io/docs/tasks/


## Install
 > Below, you will find the installation steps for GNU/Linux. For Windows and MacOS please check the docs.

With root or a sudoer user execute the following commands
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube

curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```


## Exercises
 > Below, all the commands are for GNU/Linux. Minikube and Kubectl commands should be the same on all platforms. For commands like `curl`, `vim` and `mkdir` please check the alternatives you have on other operation systems.

With your local user execute the following commands. Start the Minikube.
```
minikube start
```


### Exercise 1: hello minikube test
```
# start a deployment (in this case, it contains one pod)
kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080

# check deplyments - we should have a hello-minikube in the list 
kubectl get deployment

# check pods status - the hello-minikube-* pod should be eventually in the Running status
kubectl get pod

# check the deployment information
kubectl describe deployment hello-minikube

# check the hello-minikube-* pod information
kubectl describe pod hello-minikube

# create a service that exposes the deployment externally
kubectl expose deployment hello-minikube --type=NodePort

# check the service - there should be an hello-minikube service of type NodePort
kubectl get service

# check the service information
kubectl describe service hello-minikube

# get the service URL
minikube service hello-minikube --url

# test the app deployed
# use the URL given in your browser to check the response ...
# ... alternativelly you can use curl
curl $(minikube service hello-minikube --url)

# check the minikube dashboard
minikube dashboard

# cleanup - remove the service
kubectl delete service hello-minikube

# check the service - hello-minikube service should be missing
kubectl get service

# cleanup - remove the deployment
kubectl delete deployment hello-minikube

# check pods status - the hello-minikube-* pod should not be shown... eventually
kubectl get pod

# check deplyments status - the hello-minikube deployment should not be shown
kubectl get deployment
```


### Exercise 2: configure a deployment and a service from scratch
```
# create a directory for this exercise
mkdir ex2
cd ex2

# Part 1. NGiNX
## create a deployment file
vim nginx-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.12-alpine
        ports:
        - containerPort: 80

## start the nginx-deployment
kubectl apply --filename nginx-deployment.yaml

## check deployments
kubectl get deployment

## check the new pods created, eventually three pods should be running
kubectl get pod 

## get more info on the pods
kubectl describe pod nginx-deployment- # check the pods' IP addresses

## ssh into the Kubernetes node/host
minikube ssh

## test the app deployed on each pod
curl 172.17.0.4:80
curl 172.17.0.5:80
curl 172.17.0.6:80

## log out of the Minikube
exit

## cleanup
kubectl delete deployment nginx-deployment


# Part 2. Test-WebServer
## create a deployment file
vim test-webserver-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-webserver-deployment
  labels:
    app: test-webserver
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-webserver
  template:
    metadata:
      labels:
        app: test-webserver
    spec:
      containers:
      - name: test-webserver
        image: kubernetes/test-webserver:latest
        ports:
        - containerPort: 80

## start the test-webserver-deployment
kubectl apply --filename test-webserver-deployment.yaml

## check deployments
kubectl get deployment

## check pods
kubectl get pod

## create a service file
vim test-webserver-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: test-webserver-service
  labels:
    name: test-webserver-service
spec:
  type: NodePort
  ports:
    # the port that this service should serve on
    - port: 80
  selector:
    app: test-webserver

## start the test-webserver-service
kubectl apply --filename test-webserver-service.yaml

## check services
kubectl get service

## get service's URL
minikube service test-webserver-service --url

## check that the hostname is different running curl a few times
curl http://192.168.99.101:31633/etc/hostname
curl http://192.168.99.101:31633/etc/hostname
curl http://192.168.99.101:31633/etc/hostname
curl http://192.168.99.101:31633/etc/hostname
curl http://192.168.99.101:31633/etc/hostname

## cleanup
kubectl delete service test-webserver-service

kubectl delete deployment test-webserver-deployment
```


### Exercise 3: HTTP healthchecks using liveness
```
cd ..
# create a directory for this exercise
mkdir ex3
cd ex3

# create a pod file - the image countains an app which returns a health check status through HTTP. For the first 10 seconds that the Container is alive, the /healthz handler returns a status of 200. After that, the handler returns a status of 500.

vim liveness-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  labels:
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: kubernetes/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
      timeoutSeconds: 10
      successThreshold: 1
      failureThreshold: 3

# start the pod
kubectl create -f liveness-pod.yaml

# within 10 seconds (after the pull event), view the pod events
kubectl describe pod liveness-http

# after 10 seconds (after the pull event), view the pod events again; the liveness probe should have failed and the container should have been killed and recreated
kubectl describe pod liveness-http

# check the pod status; you should see it running sometimes and sometimes with a crash status; also the number of restarts should increment with time
kubectl get pod liveness-http

# delete the pod
kubectl delete pod liveness-http
```


### Exercise 4: namespaces, quotas, limits
```
cd ..
# create a directory for this exercise
mkdir ex4
cd ex4

# check existing namespaces
kubectl get namespaces

# create a namespace resource file - we can use YAML or JSON for the config files. In this exercise we'll use JSONs.
vim namespace-dev-with-quotas.json

{
  "apiVersion": "v1",
  "kind": "Namespace",
  "metadata": {
    "name": "dev",
    "labels": {
      "name": "development"
    }
  }
}

{
  "apiVersion": "v1",
  "kind": "ResourceQuota",
  "metadata": {
    "name": "compute-quota",
    "namespace": "dev"
  },
  "spec": {
    "hard": {
      "requests.cpu": "1",
      "requests.memory": "0.5Gi",
      "limits.cpu": "2",
      "limits.memory": "1Gi"
    }
  }
}

# create the namespace and the quota resources
kubectl create -f namespace-dev-with-quotas.json

# check existing namespaces
kubectl get namespaces --show-labels

# get more info on the new namespace created
kubectl describe namespace dev

# create a service file
vim dev-test-webserver-service.json

{
  "apiVersion": "v1",
  "kind": "Service",
  "metadata": {
    "name": "dev-test-webserver-service",
    "namespace": "dev",
    "labels": {
      "name": "development"
    }
  },
  "spec": {
    "type": "NodePort",
    "ports": [
      {
        "port": 80
      }
    ],
    "selector": {
      "app": "dev-test-webserver"
    }
  }
}

# start the service
kubectl apply --filename dev-test-webserver-service.json

# check the services - it will show up the services in the default namespace
kubectl get service

# check the services in dev namespace
kubectl get service --namespace=dev

# get all services
kubectl get service --all-namespaces

# create a deployment file
vim dev-test-webserver-deployment-no-quotas.json

{
  "apiVersion": "apps/v1",
  "kind": "Deployment",
  "metadata": {
    "name": "dev-test-webserver-deployment",
    "namespace": "dev",
    "labels": {
      "app": "dev-test-webserver"
    }
  },
  "spec": {
    "replicas": 3,
    "selector": {
      "matchLabels": {
        "app": "dev-test-webserver"
      }
    },
    "template": {
      "metadata": {
        "labels": {
          "app": "dev-test-webserver"
        }
      },
      "spec": {
        "containers": [
          {
            "name": "dev-test-webserver",
            "image": "kubernetes/test-webserver:latest",
            "ports": [
              {
                "containerPort": 80
              }
            ]
          }
        ]
      }
    }
  }
}

# start the deplyment
kubectl create -f dev-test-webserver-deployment-no-quotas.json

# check deployments in dev
kubectl get deployment --namespace=dev

# get more infor on dev-test-webserver-deployment
kubectl describe deployment dev-test-webserver-deployment --namespace=dev

# check replicasets ('rs')
kubectl get rs --namespace=dev

# get more info on our replicaset
kubectl describe rs/$(kubectl get rs --namespace=dev | tail -1 | cut -d' ' -f1) --namespace=dev

# create a deployment file
vim dev-test-webserver-deployment-with-quotas.json

{
  "apiVersion": "apps/v1",
  "kind": "Deployment",
  "metadata": {
    "name": "dev-test-webserver-deployment",
    "namespace": "dev",
    "labels": {
      "app": "dev-test-webserver"
    }
  },
  "spec": {
    "replicas": 3,
    "selector": {
      "matchLabels": {
        "app": "dev-test-webserver"
      }
    },
    "template": {
      "metadata": {
        "labels": {
          "app": "dev-test-webserver"
        }
      },
      "spec": {
        "containers": [
          {
            "name": "dev-test-webserver",
            "image": "kubernetes/test-webserver:latest",
            "ports": [
              {
                "containerPort": 80
              }
            ],
            "resources": {
              "requests": {
                "cpu": "200m",
                "memory": "0.5Gi"
              },
              "limits": {
                "cpu": "1000m",
                "memory": "1Gi"
              }
            }
          }
        ]
      }
    }
  }
}

# stop the old deployment
kubectl delete deployment dev-test-webserver-deployment --namespace=dev

# start the new deployment
kubectl create -f dev-test-webserver-deployment-with-quotas.json

# check the deployment a few times, one pod should be available
kubectl get deployment --namespace=dev

# get more info on dev-test-webserver-deployment
kubectl describe deployment dev-test-webserver-deployment --namespace=dev

# check the replicaset information
kubectl describe rs/$(kubectl get rs --namespace=dev | tail -1 | cut -d' ' -f1) --namespace=dev

# get some info on our defined compute-quota, the memory limits were reached
kubectl describe quota/compute-quota --namespace=dev

# check the dev-test-webserver-service's URL
minikube service dev-test-webserver-service --url --namespace "dev"

# checking the service, the same host should appear, as one pod is available
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname

# edit the deployment for each pod to use less memory; the end result should be something like this
#        resources:
#          limits:
#            cpu: "1"
#            memory: 300Mi
#          requests:
#            cpu: 200m
#            memory: 100Mi
kubectl edit deployment dev-test-webserver-deployment --namespace=dev

# scale down the dev-test-webserver-deployment
kubectl scale deployment dev-test-webserver-deployment --replicas=0 --namespace=dev

# scale down the deployment to its initial value
kubectl scale deployment dev-test-webserver-deployment --replicas=3 --namespace=dev

# check the deployment a few times, two pods should be available
kubectl get deployment --namespace=dev

# checking quota, we can see the cpu limits were reached
kubectl describe quota/compute-quota --namespace=dev

# checking the service, two hosts should appear, as two pods are available
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname

# edit the deployment for each pod to use less cpu; the end result should be something like this
#        resources:
#          limits:
#            cpu: 600m
#            memory: 300Mi
#          requests:
#            cpu: 200m
#            memory: 100Mi
kubectl edit deployment dev-test-webserver-deployment --namespace=dev

# scale down the dev-test-webserver-deployment
kubectl scale deployment dev-test-webserver-deployment --replicas=0 --namespace=dev

# scale down the deployment to its initial value
kubectl scale deployment dev-test-webserver-deployment --replicas=3 --namespace=dev

# check the deployment a few times, all pods should be available
kubectl get deployment --namespace=dev

# check the pods
kubectl get pods --namespace=dev

# check the quotas
kubectl describe quota/compute-quota --namespace=dev

# checking the service, all three hosts should appear
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname
curl $(minikube service dev-test-webserver-service --url --namespace "dev")/etc/hostname

# cleanup
kubectl delete deployment dev-test-webserver-deployment --namespace=dev

kubectl delete service dev-test-webserver-service --namespace=dev

kubectl delete quota/compute-quota --namespace=dev

kubectl delete namespace dev

kubectl get namespaces
```

Stop the Minikube.
```
minikube stop
```
