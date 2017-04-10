# K8S cluster

Educational project about building Kubernetes cluster inside of VirtualBox VMs.

[![Build Status](https://travis-ci.org/gongled/kubernetes-cluster.svg?branch=master)](https://travis-ci.org/gongled/kubernetes-cluster)

## Requirements

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 5.1.0 or higher
- [Vagrant](https://www.vagrantup.com/downloads.html) 1.9.0 or higher
- [Vagrant VBGuest plug-in](https://github.com/dotless-de/vagrant-vbguest) 0.13.0 or higher
- [Ansible](https://docs.ansible.com/ansible/intro_installation.html#installation) 2.0 or higher

## Getting started

Specify amount of CPUs, RAM and role in cluster in `config.yml`.

_Note: If you are going to change hostname of the master node, keep in mind that Ansible parameters are available in [group_vars/all](https://github.com/gongled/kubernetes-cluster/blob/master/provision/ansible/group_vars/all) configuration file._

```
vms:
  master0:
    box: "centos/7"
    ip: 10.0.4.10
    cpus: 2
    memory: 1024
    role: master
    shared:
      - "containers:/containers"
  slave0:
    box: "centos/7"
    ip: 10.0.4.11
    cpus: 2
    memory: 1024
    role: slave
  slave1:
    box: "centos/7"
    ip: 10.0.4.12
    cpus: 2
    memory: 1024
    role: slave
  slave2:
    box: "centos/7"
    ip: 10.0.4.13
    cpus: 2
    memory: 1024
    role: slave

provision:
  ansible:
    playbook: "provision/ansible/playbook.yml"
```

Run `vagrant up` to launch VMs cluster based on VirtualBox.

Well done. It works.

## Components

- Master
  - Kubernetes API Server
  - Kubernetes Controller Manager
  - Kubernetes Scheduler
  - Kubernetes Client (kubectl) 
  - flanneld
  - etcd
- Node(s) 
  - Kubernetes Node
  - Kubernetes Proxy
  - flanneld

## Run an example service

Connect to the master VM and execute the following commands:

1. Create a deployment

    ```
    $ kubectl apply -f /containers/nginx-deployment.yml
    ```

2. Create service to expose pods port

    ```
    $ kubectl apply -f /containers/nginx-service.yml
    ```

3. Inspect `nginx` service configuration

    ```
    $ kubectl describe services/nginx
    Name:			nginx
    Namespace:		default
    Labels:			<none>
    Selector:		app=nginx
    Type:			NodePort
    IP:			10.254.245.228
    Port:			<unset>	80/TCP
    NodePort:		<unset>	30001/TCP
    Endpoints:		172.17.103.2:80,172.17.73.2:80,172.17.94.2:80
    Session Affinity:	None
    ```

4. Test your application

    ```
    $ curl -I http://slave0:30001/
    HTTP/1.1 200 OK
    Server: nginx/1.7.9
    Date: Sun, 09 Apr 2017 20:18:37 GMT
    Content-Type: text/html
    Content-Length: 612
    Last-Modified: Tue, 23 Dec 2014 16:25:09 GMT
    Connection: keep-alive
    ETag: "54999765-264"
    Accept-Ranges: bytes
    ```

## License

MIT
