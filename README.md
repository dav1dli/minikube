# Install Minikube with Ansible
This playbook installs minikube and dependencies with Ansible on a localhost. It is an example. It can be extended to support multiple flavors of Linux and other operating systems.

Minikube on Linux supports KVM or VirtualBox hypervisors. For simplicity purposes it is assumed that KVM is used.

## Prerequisites

* Virtualization enabled in BIOS
* SUDO permissions for the current user
* Linux (RHEL, CentOS, Fedora)
* python3
* python3-pip
* @virt package group, virt-top libguestfs-tools
* ansible

## Installation steps

### Virtualization

Minikube creates and runs a VM. For which CPU support for virtualization in BIOS must be enabled. To verify: `grep -E 'vmx|svm' /proc/cpuinfo`

The playbook makes sure that required hyprvisor components are installed and services are running. It also enables the current user to manipulate VMs.
 
### Ansible

On RHEL with enabled subscription Ansible is provided from ansible repository disabled by default.

```
sudo subscription-manager repos --enable ansible-2.8-for-rhel-8-x86_64-rpms
sudo yum -y install ansible
```

On CentOS Ansible can be installed using python-pip:
```
sudo yum -y install python3-pip
pip3 install ansible --user
```

Those tasks are not automated in provided playbooks. 

Verify ansible: `ansible localhost -m ping`

### Kubernetes tools
See [Kubernetes Install Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/), [Minikube project page](https://github.com/kubernetes/minikube/) and [the documentation](https://minikube.sigs.k8s.io/docs/) for more information.

All Kubernetes components, including the CLI, minikube and helm package manager are downloaded from the Internet and installed locally in the path.

Minikube configuration is created in the home directory of the current user.

Kubectl CLI is configured to work with this configuration. Verify: `kubectl cluster-info`

#### Troublesooting
If a previous configuration is stuck and causes errors it is possible to reinitialize the environment by deleting the k8s cluster with `minikube delete --all`

### Jenkins

JenkinsCI is installed using an official stable Helm chart: `helm install jenkins stable/jenkins`

Default user: `admin`

Password: `kubectl get secret --namespace default jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode`

To access the GUI:
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" \
	-l "app.kubernetes.io/instance=jenkins" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace default port-forward $POD_NAME 8080:8080
```
Login at http://127.0.0.1:8080 with user `admin` and the password obtained above.