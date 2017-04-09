# K8S Cluster

Educational project about building Kubernetes cluster inside of VirtualBox VMs.

## Requirements

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 5.1.0 or higher
- [Vagrant](https://www.vagrantup.com/downloads.html) 1.9.0 or higher
- [Vagrant VBGuest plug-in](https://github.com/dotless-de/vagrant-vbguest) 0.13.0 or higher
- [Ansible](https://docs.ansible.com/ansible/intro_installation.html#installation) 2.0 or higher

## Getting started

Specify amount of CPUs, RAM and role in cluster in `config.yml` file:

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

Done.

## Run a test service

Connect to VM and execute the following commands: 

1. Create a deployment

```
kubectl apply -f /containers/nginx-deployment.yml
```

2. Expose port to the service

```
kubectl expose deployments/nginx-deployment --type="NodePort" --port 80
```

## License

MIT

