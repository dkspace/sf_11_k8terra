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

# to delete all applied  
>terraform destroy
>yes

```
# Practicum
## Yandex Cloud 
[Начало работы с Terraform](https://cloud.yandex.ru/docs/solutions/infrastructure-management/terraform-quickstart#check-resources)

[YandexCloud CLI](https://cloud.yandex.ru/docs/cli/quickstart)

[compute_instance parameter description](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/compute_instance)

    yc init

```shell 
main.tf
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  token     = "<OAuth>"
  cloud_id  = "<идентификатор облака>"
  folder_id = "<идентификатор каталога>"
  zone      = "ru-central1-a"
}

resource "yandex_compute_instance" "vm-1" {
  name = "terraform1"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd87va5cc00gaq2f5qfb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }

  metadata = {
    #    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
    user-data = "${file("~/sf_module_11/yandexCloud/meta1.txt")}"
  }
}

resource "yandex_compute_instance" "vm-2" {
  name = "terraform2"

  resources {
    cores  = 4
    memory = 4
  }

  boot_disk {
    initialize_params {
      image_id = "fd87va5cc00gaq2f5qfb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }

  metadata = {
    #    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
    user-data = "${file("~/sf_module_11/yandexCloud/meta2.txt")}"
  }
}

resource "yandex_vpc_network" "network-1" {
  name = "network1"
}

resource "yandex_vpc_subnet" "subnet-1" {
  name           = "subnet1"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network-1.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}

output "internal_ip_address_vm_1" {
  value = yandex_compute_instance.vm-1.network_interface.0.ip_address
}

output "internal_ip_address_vm_2" {
  value = yandex_compute_instance.vm-2.network_interface.0.ip_address
}


output "external_ip_address_vm_1" {
  value = yandex_compute_instance.vm-1.network_interface.0.nat_ip_address
}

output "external_ip_address_vm_2" {
  value = yandex_compute_instance.vm-2.network_interface.0.nat_ip_address
}

#Create meta.txt
----------------------------------------------------------
#cloud-config
users:
  - name: <имя пользователя>
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa AAAAB3Nza......OjbSMRX user@example.com
      - ssh-rsa AAAAB3Nza......Pu00jRN user@desktop
----------------------------------------------------------      
In main.tf change ssh-keys with user-data 

metadata = {
    user-data = "${file("<путь к файлу>/meta.txt")}"
}

```

```shell
>terraform init
>terraform validate
>terraform fmt
>ls -la
main.tf
variables.tf
>terraform plan

>terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_compute_instance.vm-1 will be created
  + resource "yandex_compute_instance" "vm-1" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + hostname                  = (known after apply)
      + id                        = (known after apply)
      + metadata                  = {
          + "ssh-keys" = <<-EOT
                ubuntu:ssh-rsa AAAA...... dmik@dkhost
            EOT
        }
      + name                      = "terraform1"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + description = (known after apply)
              + image_id    = "fd87va5cc00gaq2f5qfb"
              + name        = (known after apply)
              + size        = (known after apply)
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy {
          + placement_group_id = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 2
        }

      + scheduling_policy {
          + preemptible = (known after apply)
        }
    }

  # yandex_compute_instance.vm-2 will be created
  + resource "yandex_compute_instance" "vm-2" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + hostname                  = (known after apply)
      + id                        = (known after apply)
      + metadata                  = {
          + "ssh-keys" = <<-EOT
                ubuntu:ssh-rsa AAAA....... dmik@dkhost
            EOT
        }
      + name                      = "terraform2"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + description = (known after apply)
              + image_id    = "fd87va5cc00gaq2f5qfb"
              + name        = (known after apply)
              + size        = (known after apply)
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy {
          + placement_group_id = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 4
          + memory        = 4
        }

      + scheduling_policy {
          + preemptible = (known after apply)
        }
    }

  # yandex_vpc_network.network-1 will be created
  + resource "yandex_vpc_network" "network-1" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + name                      = "network1"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.subnet-1 will be created
  + resource "yandex_vpc_subnet" "subnet-1" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + name           = "subnet1"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.10.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

Plan: 4 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + external_ip_address_vm_1 = (known after apply)
  + external_ip_address_vm_2 = (known after apply)
  + internal_ip_address_vm_1 = (known after apply)
  + internal_ip_address_vm_2 = (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

yandex_vpc_network.network-1: Creating...
yandex_vpc_network.network-1: Creation complete after 2s [id=enpdk6dvg7i7huna8chg]
yandex_vpc_subnet.subnet-1: Creating...
yandex_vpc_subnet.subnet-1: Creation complete after 1s [id=e9b3tmdt15g1s85371or]
yandex_compute_instance.vm-2: Creating...
yandex_compute_instance.vm-1: Creating...
yandex_compute_instance.vm-2: Still creating... [10s elapsed]
yandex_compute_instance.vm-1: Still creating... [10s elapsed]
yandex_compute_instance.vm-2: Still creating... [20s elapsed]
yandex_compute_instance.vm-1: Still creating... [20s elapsed]
yandex_compute_instance.vm-2: Creation complete after 27s [id=fhmgsk5io9q61lotsogp]
yandex_compute_instance.vm-1: Creation complete after 28s [id=fhmhbsrv2plide4a0h1v]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

external_ip_address_vm_1 = "178.154.246.192"
external_ip_address_vm_2 = "178.154.202.197"
internal_ip_address_vm_1 = "192.168.10.17"
internal_ip_address_vm_2 = "192.168.10.6"

```
```sell
>yc compute instance list
+----------------------+------------+---------------+---------+-----------------+---------------+
|          ID          |    NAME    |    ZONE ID    | STATUS  |   EXTERNAL IP   |  INTERNAL IP  |
+----------------------+------------+---------------+---------+-----------------+---------------+
| epdf9vld1ugohgr250hl | dkhost     | ru-central1-b | STOPPED |                 | 10.129.0.20   |
| fhmgsk5io9q61lotsogp | terraform2 | ru-central1-a | RUNNING | 178.154.202.197 | 192.168.10.6  |
| fhmhbsrv2plide4a0h1v | terraform1 | ru-central1-a | RUNNING | 178.154.246.192 | 192.168.10.17 |
+----------------------+------------+---------------+---------+-----------------+---------------+
>ssh dmik@178.154.202.197
>ssh dmik@178.154.246.192
```
### Kuberadm to create cluster
[Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

[Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

```shell
>sudo modprobe br_netfilter
>sudo vi /etc/sysctl.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

>sudo sysctl -p
```
### Install containerd: Docker Engine on Debian
[install-using-the-convenience-script](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script)

```shell
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

# Install

>curl -fsSL https://get.docker.com -o get-docker.sh
>sudo sh get-docker.sh
>docker -v

sudo groupadd docker
sudo usermod -aG docker $USER
sudo newgrp docker
#check use docker w/o sudo
>docker run hello-world
#in case problem use https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
sudo systemctl enable docker.service
sudo systemctl enable containerd.service

#configure containerd
sudo mkdir -p /etc/containerd
sudo systemctl restart containerd
```
### Installing kubeadm, kubelet and kubectl

```shell
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
```shell
>kubeadm init
[init] Using Kubernetes version: v1.21.1
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at h

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.10.30:6443 --token 0rh9ri.9k1fvsb15b8hfa9q \
	--discovery-token-ca-cert-hash sha256:bbd924527e1a1953aaeef7590e04d107cb1a073606ba76f05bb06f44c8afd109 

>kubectl get pods --all-namespaces
NAMESPACE     NAME                                           READY   STATUS    RESTARTS   AGE
kube-system   coredns-558bd4d5db-2st7h                       0/1     Pending   0          14m
kube-system   coredns-558bd4d5db-vff76                       0/1     Pending   0          14m
kube-system   etcd-fhmlmpfe9amtmjb96r3b                      1/1     Running   0          14m
kube-system   kube-apiserver-fhmlmpfe9amtmjb96r3b            1/1     Running   0          14m
kube-system   kube-controller-manager-fhmlmpfe9amtmjb96r3b   1/1     Running   0          14m
kube-system   kube-proxy-422f4                               1/1     Running   0          14m
kube-system   kube-scheduler-fhmlmpfe9amtmjb96r3b            1/1     Running   0          14m

```




