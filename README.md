# Kubernetas as orchestrator

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

```
