# Kubernetas = orchestrator

This is an exercise for DevOps training from [SkillFactory.ru - DEVOPS](https://lms.skillfactory.ru/)

## How To instal minikube

### to install Minikube

Prepare system with installed kubectl : [Install and setup kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/#%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-kubectl-%D0%B2-linux)
```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl
kubectl version
```
Also [VirtualBox](https://kubernetes.io/ru/docs/setup/learning-environment/minikube/#%D1%83%D0%BA%D0%B0%D0%B7%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B4%D1%80%D0%B0%D0%B9%D0%B2%D0%B5%D1%80%D0%B0-%D0%B2%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B9-%D0%BC%D0%B0%D1%88%D0%B8%D0%BD%D1%8B) should be installed

[Install minikube](https://kubernetes.io/ru/docs/tasks/tools/install-minikube/)
Use [git hub releases](https://github.com/kubernetes/minikube/releases)

or use :
```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
Чтобы исполняемый файл Minikube был доступен из любой директории выполните следующие команды:

sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/

# For Checking >minikube start --vm-driver=<driver_name> 
#driver_name = virtualbox ,vmwarefusion , docker , kvm2 , hyperkit , hyperv 

#to start
minikube start --vm-driver=virtualbox

minikube status
#to stop

minikube stop
#before new start it need to clear local status
minikube delete

```
## Operation with K8s 
[k8s cheatcheet](https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/)
```shell
>kubectl get pods --all-namespaces
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-wbssc            1/1     Running   0          17m
kube-system   etcd-minikube                      1/1     Running   0          17m
kube-system   kube-apiserver-minikube            1/1     Running   0          17m
kube-system   kube-controller-manager-minikube   1/1     Running   0          17m
kube-system   kube-proxy-4lkwq                   1/1     Running   0          17m
kube-system   kube-scheduler-minikube            1/1     Running   0          17m
kube-system   storage-provisioner                1/1     Running   1          18m
```
### Creating some resouce with minikube

```shell
>kubectl create deployment hello-app --image=k8s.gcr.io/echoserver:1.4
deployment.apps/hello-app created

>kubectl get deployments --all-namespaces
NAMESPACE     NAME        READY   UP-TO-DATE   AVAILABLE   AGE
default       hello-app   0/1     1            0           14s
kube-system   coredns     1/1     1            1           21m

```
### Publishing resoгrce to outside

```shell
# requred the service type= NodePort :

>kubectl expose deployment hello-app --type=NodePort --port=8080
service/hello-app exposed
# to check
>kubectl get services hello-app
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-app   NodePort   10.98.149.162   <none>        8080:31044/TCP   58s

# to porward port of the service to localhost 
>kubectl port-forward service/hello-app 7080:8080
Forwarding from 127.0.0.1:7080 -> 8080
Forwarding from [::1]:7080 -> 8080
```
From now we can open link http://localhost:7080 and get in the browser responce :
```
CLIENT VALUES:
client_address=127.0.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://localhost:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
accept-encoding=gzip, deflate, br
accept-language=ru-RU,ru;q=0.9,en-US;q=0.8,en;q=0.7,pt;q=0.6
connection=keep-alive
host=localhost:7080
sec-ch-ua="Chromium";v="88", "Google Chrome";v="88", ";Not A Brand";v="99"
sec-ch-ua-mobile=?0
sec-fetch-dest=document
sec-fetch-mode=navigate
sec-fetch-site=cross-site
sec-fetch-user=?1
upgrade-insecure-requests=1
user-agent=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.182 Safari/537.36
BODY:
-no body in request-
```
## To deploy nginx into cluster using yaml file

Create deployment.yaml which describes this deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # мы создадим 2 реплики nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
and appy this deployment file
```shell
>kubectl apply -f deployment.yaml
deployment.apps/nginx-deployment created

#To check the result serch all PODs with filtering by the lable
>kubectl get pods -l app=nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-66b6c48dd5-qp76r   1/1     Running   0          3m26s
nginx-deployment-66b6c48dd5-t9h44   1/1     Running   0          3m26s

```
### update deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # Меняем количество реплик
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16.1 # Меняем версию образа
        ports:
        - containerPort: 80
```

```shell
>kubectl apply -f deployment.yaml
deployment.apps/nginx-deployment configured
>kubectl get pods -l app=nginx
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-559d658b74-dvj2j   0/1     ContainerCreating   0          7s
nginx-deployment-559d658b74-sfndq   0/1     ContainerCreating   0          7s
nginx-deployment-66b6c48dd5-lks8c   1/1     Running             0          7s
nginx-deployment-66b6c48dd5-qp76r   1/1     Running             0          6m32s
nginx-deployment-66b6c48dd5-t9h44   1/1     Running             0          6m32s
nginx-deployment-66b6c48dd5-xj276   0/1     Terminating         0          7s
(base) [dmik@dkhost sf_module_11]$ 

```

## resource managment

```shell
>kubectl run -i --tty --rm busybox --image=busybox --restart=Never --requests='cpu=50m,memory=50Mi' -- sh
Flag --requests has been deprecated, has no effect and will be removed in the future.
If you don't see a command prompt, try pressing enter.

# but trainer not experienced with this command ^^ 
```
[Managing Resources for Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

[Configure Minimum and Maximum CPU Constraints for a Namespace](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/)

[Задание ресурсов CPU для контейнеров и Pod'ов](https://kubernetes.io/ru/docs/tasks/configure-pod-container/assign-cpu-resource/)

#### Launch server of metrics for Minikube
```shell
>minikube addons enable metrics-server
# check result
>kubectl get apiservices | grep metrics.k8s.io
v1beta1.metrics.k8s.io                 kube-system/metrics-server   True        3m24s

```

#### Create a namespace so that the resources you create in this exercise are isolated from the rest of your cluster.
```shell
>kubectl create namespace constraints-cpu-example
```
#### Create cpu-request-limit.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: demo
spec:
  containers:
  - name: busybox        
    image: busybox
    resources:
      requests:
        memory: "50Mi"
        cpu: "50m"
```
```shell
>kubectl apply -f cpu-request-limit.yaml --namespace=constraints-cpu-example
pod/demo created

>kubectl get pod busybox --output=yaml --namespace=constraints-cpu-example
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"demo","namespace":"constraints-cpu-example"},"spec":{"containers":[{"image":"busybox","name":"busybox","resources":{"requests":{"cpu":"50m","memory":"50Mi"}}}]}}
  creationTimestamp: "2021-05-20T23:34:31Z"
  name: demo
  namespace: constraints-cpu-example
  resourceVersion: "3088"
  uid: 6beafd85-ad55-4510-914b-27279cc9a35c
spec:
  containers:
  - image: busybox
    imagePullPolicy: Always
    name: busybox
    resources:
      requests:
        cpu: 50m
        memory: 50Mi
...


ubectl get pods --all-namespaces
kubectl get pods --namespace=constraints-cpu-example
kubectl delete pods demo --namespace=constraints-cpu-example
kubectl apply -f cpu-request-limit.yaml --namespace=constraints-cpu-example
kubectl exec -it demo -- /bin/bash
```

## How To instal Terraform 
[Install Terraform] (https://learn.hashicorp.com/tutorials/terraform/install-cli)
```shell
sudo yum install -y yum-utils
yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
yum -y install terraform
```
### How To use Terraform

```shell
mkdir terraform-docker-demo && cd $_
touch main.tf

cat main.tf
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}


>terraform init
>terraform apply
>yes

>docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
028298c44207   7ce4f91ef623   "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8000->80/tcp   tutorial

```
