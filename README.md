From following along "Kubernetes In Action" by Marco Luksa

# Setup

 - Install Docker: https://docs.docker.com/engine/install/ubuntu/    
 - Create a DockerHub account (my username there is beefybeefy)  
 - Create a google cloud account/project, enable billing, and enable Google Kubernetes Engine  
 - Install the gcloud cli: https://cloud.google.com/sdk/docs/install  
 - Install the kubectl cli: `gcloud components install kubectl`  

## Build the container image & push it to DockerHub

```
docker build -t kubia .
docker tag kubia beefybeefy/kubia
docker login
docker push beefybeefy/kubia
```

## Create a K8 cluster with 3 worker nodes

```
gcloud auth login
gcloud config set project nate-k8-hello-world
gcloud container clusters create kubia --num-nodes 3 --region europe-west2-a --machine-type e2-small
```

## Deploy the DockerHub image to google cloud

```
kubectl run kubia --image=beefybeefy/kubia --port=8080
```

## Create a "replication controller"/deployment with 3 replicas

```
kubectl delete pod kubia
kubectl create deployment kubia --image=beefybeefy/kubia
kubectl expose deployment kubia --type=LoadBalancer --name kubia-http --port=8080
kubectl scale deployment kubia --replicas=3
```

## Make requests to the replicas and check that they are being load balanced randomly

```
nate@spooky:~/k8_hello_world$ kubectl get services
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
kubernetes   ClusterIP      xx.xx.x.x      <none>           443/TCP          22m
kubia-http   LoadBalancer   xx.xx.xx.xxx   xx.xxx.xxx.xxx   8080:30282/TCP   92s
nate@spooky:~/k8_hello_world$ curl <external-IP>:8080
Nate says 'Hello World' from kubia-848cf6987f-xhcs7
nate@spooky:~/k8_hello_world$ curl <external-IP>:8080
Nate says 'Hello World' from kubia-848cf6987f-pgphc
nate@spooky:~/k8_hello_world$ curl <external-IP>:8080
Nate says 'Hello World' from kubia-848cf6987f-27kvc
```

# Cleanup when done

```
kubectl delete service kubia-http
kubectl delete deployment kubia
gcloud container delete cluster kubia
```
