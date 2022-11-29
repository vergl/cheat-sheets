# Kubespray
Kubespray is a composition of Ansible playbooks, inventory, provisioning tools, and domain knowledge for generic OS/Kubernetes clusters configuration management tasks.

## Prerequisites

[Python 3](https://www.python.org/downloads/), [pip3](https://pip.pypa.io/en/stable/) and [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) should be installed.
```bash
sudo apt install python3 python3-pip ansible
```

### Generate SSH key and install it to nodes
First, we need to generate SSH keys (if they are not generated yet). All default values on prompt should be fine in our case
```bash
ssh-keygen
```

Next, we need to pass our public key to VMs which we are going to use. Example for our first master node:
```bash
ssh-copy-id 192.168.0.131
```

### Clone Kubespray repository
Next, clone Kubespray using git and navigate into this directory
```bash
git clone https://github.com/kubernetes-sigs/kubespray.git && cd kubespray
```

*Optional:* we can change git branch to the latest tag (current latest: v2.20.0)
```bash
tag=$(git describe --tags `git rev-list --tags --max-count=1`)
git checkout $tag -b latest
```

### Install required packages
To install required packages we can use *pip* package manager
```bash
pip3 install -r requirements.txt
```

### Copy sample inventory
Copy sample inventory to a different directory where we can make some changes:
```bash
cp -rfp inventory/sample inventory/myk8s
```

### Generate inventory files
Now we can generate inventory files with built-in tool *inventory_builder*.

First, we need to specify IP addresses of our VMs:
```bash
declare -a IPS=(192.168.0.131 192.168.0.132 192.168.0.133 192.168.0.141 192.168.0.142 192.168.0.143)
```

And now we a ready to generate our own hosts file:
```bash
CONFIG_FILE=inventory/myk8s/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

Generated `inventory/myk8s/hosts.yaml` file:
```yaml
all:
  hosts:
    node1:
      ansible_host: 192.168.0.131
      ip: 192.168.0.131
      access_ip: 192.168.0.131
    node2:
      ansible_host: 192.168.0.132
      ip: 192.168.0.132
      access_ip: 192.168.0.132
    node3:
      ansible_host: 192.168.0.133
      ip: 192.168.0.133
      access_ip: 192.168.0.133
    node4:
      ansible_host: 192.168.0.141
      ip: 192.168.0.141
      access_ip: 192.168.0.141
    node5:
      ansible_host: 192.168.0.142
      ip: 192.168.0.142
      access_ip: 192.168.0.142
    node6:
      ansible_host: 192.168.0.143
      ip: 192.168.0.143
      access_ip: 192.168.0.143
  children:
    kube_control_plane:
      hosts:
        node1:
        node2:
    kube_node:
      hosts:
        node1:
        node2:
        node3:
        node4:
        node5:
        node6:
    etcd:
      hosts:
        node1:
        node2:
        node3:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

Let's rename our nodes and make 3 of them master nodes as we initially decided. The result should look like this:
```yaml
all:
  hosts:
    master1:
      ansible_host: 192.168.0.131
      ip: 192.168.0.131
      access_ip: 192.168.0.131
    master2:
      ansible_host: 192.168.0.132
      ip: 192.168.0.132
      access_ip: 192.168.0.132
    master3:
      ansible_host: 192.168.0.133
      ip: 192.168.0.133
      access_ip: 192.168.0.133
    worker1:
      ansible_host: 192.168.0.141
      ip: 192.168.0.141
      access_ip: 192.168.0.141
    worker2:
      ansible_host: 192.168.0.142
      ip: 192.168.0.142
      access_ip: 192.168.0.142
    worker3:
      ansible_host: 192.168.0.143
      ip: 192.168.0.143
      access_ip: 192.168.0.143
  children:
    kube_control_plane:
      hosts:
        master1:
        master2:
        master3:
    kube_node:
      hosts:
        worker1:
        worker2:
        worker3:
    etcd:
      hosts:
        master1:
        master2:
        master3:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

We can also review and change parameters under `inventory/mycluster/group_vars`, but this time we're going to use default settings:
```bash
cat inventory/myk8s/group_vars/all/all.yml
cat inventory/myk8s/group_vars/k8s_cluster/k8s-cluster.yml
```

### Install Kubernetes to our nodes
The final part, let's install Kubernetes using `Ansible` and `Kubespray`:
```bash
ansible-playbook -i inventory/myk8s/hosts.yaml  --become --become-user=root cluster.yml
```

### Copy kubectl configuration
Now when our cluster is ready we can get our configuration file to connect to cluster from our PC.
```bash
ssh 192.168.0.131
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
exit
scp 192.168.0.131:~/.kube/config ~/.kube/config
```

and we should change line
```bash
...
    server: https://127.0.0.1:6443
...
```

to:
```bash
...
    server: https://192.168.0.131:6443
...
```

### Using new shiny k8s cluster
The cluster is up and running, let's check which node are available:
```bash
kubectl get nodes -o wide

NAME      STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master1   Ready    control-plane   26m   v1.24.6   192.168.0.131   <none>        Ubuntu 22.04.1 LTS   5.15.0-53-generic   containerd://1.6.8
master2   Ready    control-plane   25m   v1.24.6   192.168.0.132   <none>        Ubuntu 22.04.1 LTS   5.15.0-53-generic   containerd://1.6.8
master3   Ready    control-plane   25m   v1.24.6   192.168.0.133   <none>        Ubuntu 22.04.1 LTS   5.15.0-53-generic   containerd://1.6.8
worker1   Ready    <none>          24m   v1.24.6   192.168.0.141   <none>        Ubuntu 22.04.1 LTS   5.15.0-53-generic   containerd://1.6.8
worker2   Ready    <none>          24m   v1.24.6   192.168.0.142   <none>        Ubuntu 22.04.1 LTS   5.15.0-53-generic   containerd://1.6.8
worker3   Ready    <none>          24m   v1.24.6   192.168.0.143   <none>        Ubuntu 22.04.1 LTS   5.15.0-53-generic   containerd://1.6.8
```

And which pods are currently running:
```bash
kubectl get pods -o wide --all-namespaces

NAMESPACE     NAME                              READY   STATUS    RESTARTS        AGE   IP              NODE      NOMINATED NODE   READINESS GATES
kube-system   calico-node-5528r                 1/1     Running   1 (7m2s ago)    26m   192.168.0.132   master2   <none>           <none>
kube-system   calico-node-6vrzr                 1/1     Running   1 (10m ago)     26m   192.168.0.142   worker2   <none>           <none>
kube-system   calico-node-7gslp                 1/1     Running   1 (14m ago)     26m   192.168.0.141   worker1   <none>           <none>
kube-system   calico-node-b5cm9                 1/1     Running   1 (8m39s ago)   26m   192.168.0.131   master1   <none>           <none>
kube-system   calico-node-bzgkt                 1/1     Running   1 (5m41s ago)   26m   192.168.0.133   master3   <none>           <none>
kube-system   calico-node-rqtnq                 1/1     Running   1 (9m56s ago)   26m   192.168.0.143   worker3   <none>           <none>
kube-system   coredns-74d6c5659f-mb7c9          1/1     Running   1 (5m41s ago)   25m   10.233.72.2     master3   <none>           <none>
kube-system   coredns-74d6c5659f-wfkpd          1/1     Running   1 (8m39s ago)   25m   10.233.104.66   master1   <none>           <none>
kube-system   dns-autoscaler-59b8867c86-kggtp   1/1     Running   1 (7m2s ago)    25m   10.233.116.2    master2   <none>           <none>
kube-system   kube-apiserver-master1            1/1     Running   2 (8m39s ago)   28m   192.168.0.131   master1   <none>           <none>
kube-system   kube-apiserver-master2            1/1     Running   2 (7m2s ago)    28m   192.168.0.132   master2   <none>           <none>
kube-system   kube-apiserver-master3            1/1     Running   2 (5m42s ago)   27m   192.168.0.133   master3   <none>           <none>
kube-system   kube-controller-manager-master1   1/1     Running   2 (8m39s ago)   28m   192.168.0.131   master1   <none>           <none>
kube-system   kube-controller-manager-master2   1/1     Running   2 (7m2s ago)    27m   192.168.0.132   master2   <none>           <none>
kube-system   kube-controller-manager-master3   1/1     Running   3 (5m42s ago)   27m   192.168.0.133   master3   <none>           <none>
kube-system   kube-proxy-258hh                  1/1     Running   1 (10m ago)     26m   192.168.0.142   worker2   <none>           <none>
kube-system   kube-proxy-667j9                  1/1     Running   1 (7m2s ago)    26m   192.168.0.132   master2   <none>           <none>
kube-system   kube-proxy-7zhmr                  1/1     Running   1 (8m39s ago)   26m   192.168.0.131   master1   <none>           <none>
kube-system   kube-proxy-d9f8c                  1/1     Running   1 (5m41s ago)   26m   192.168.0.133   master3   <none>           <none>
kube-system   kube-proxy-kf6tn                  1/1     Running   1 (9m56s ago)   26m   192.168.0.143   worker3   <none>           <none>
kube-system   kube-proxy-tww6d                  1/1     Running   1 (14m ago)     26m   192.168.0.141   worker1   <none>           <none>
kube-system   kube-scheduler-master1            1/1     Running   2 (8m39s ago)   28m   192.168.0.131   master1   <none>           <none>
kube-system   kube-scheduler-master2            1/1     Running   2 (7m2s ago)    27m   192.168.0.132   master2   <none>           <none>
kube-system   kube-scheduler-master3            1/1     Running   2 (5m42s ago)   27m   192.168.0.133   master3   <none>           <none>
kube-system   nginx-proxy-worker1               1/1     Running   1 (14m ago)     25m   192.168.0.141   worker1   <none>           <none>
kube-system   nginx-proxy-worker2               1/1     Running   1 (10m ago)     25m   192.168.0.142   worker2   <none>           <none>
kube-system   nginx-proxy-worker3               1/1     Running   1 (9m56s ago)   25m   192.168.0.143   worker3   <none>           <none>
kube-system   nodelocaldns-644mk                1/1     Running   1 (9m56s ago)   25m   192.168.0.143   worker3   <none>           <none>
kube-system   nodelocaldns-dgtzd                1/1     Running   1 (7m2s ago)    25m   192.168.0.132   master2   <none>           <none>
kube-system   nodelocaldns-lttvl                1/1     Running   1 (10m ago)     25m   192.168.0.142   worker2   <none>           <none>
kube-system   nodelocaldns-rxwjm                1/1     Running   1 (14m ago)     25m   192.168.0.141   worker1   <none>           <none>
kube-system   nodelocaldns-wdj4v                1/1     Running   1 (5m42s ago)   25m   192.168.0.133   master3   <none>           <none>
kube-system   nodelocaldns-whjlm                1/1     Running   1 (8m39s ago)   25m   192.168.0.131   master1   <none>           <none>
```

That's it, the cluster is ready to use, now we can install everything we need.

## Working with existing Kubernetes cluster
To upgrade Kubernetes cluster (upgrate to a newer version, add new node, etc) we can update values in our `inventory/myk8s` directory and run `ansible-playbook` again. It will check these files and update the cluster according to the changes.

### Configuring Kubernetes cluster
To enable [Kubernetes Dashboard](https://github.com/kubernetes/dashboard) we need to change the value in file `inventory/myk8s/group_vars/k8s_cluster/addons.yml` from
```bash
# dashboard_enabled: false
```
to
```bash
dashboard_enabled: true
```

Also, let's enable Local [Volume Provisioner](https://github.com/kubernetes-sigs/kubespray/blob/master/docs/kubernetes-apps/local_volume_provisioner.md) and [Cert Manager](https://cert-manager.io/)
```bash
local_volume_provisioner_enabled: true
...
cert_manager_enabled: true
```

and run `ansible-playbook` with this command (the last parameter is pointing to a `upgrade-cluster.yaml` file)
```bash
ansible-playbook -i inventory/myk8s/hosts.yaml  --become --become-user=root upgrade-cluster.yml
```

### Adding more worker nodes to cluster
To add more nodes to cluster we can manually add them to `inventory/myk8s/hosts.yaml` file. Let's add two more worker nodes:
```yaml
all:
  hosts:
    master1:
      ansible_host: 192.168.0.131
      ip: 192.168.0.131
      access_ip: 192.168.0.131
    master2:
      ansible_host: 192.168.0.132
      ip: 192.168.0.132
      access_ip: 192.168.0.132
    master3:
      ansible_host: 192.168.0.133
      ip: 192.168.0.133
      access_ip: 192.168.0.133
    worker1:
      ansible_host: 192.168.0.141
      ip: 192.168.0.141
      access_ip: 192.168.0.141
    worker2:
      ansible_host: 192.168.0.142
      ip: 192.168.0.142
      access_ip: 192.168.0.142
    worker3:
      ansible_host: 192.168.0.143
      ip: 192.168.0.143
      access_ip: 192.168.0.143
    worker4:
      ansible_host: 192.168.0.144
      ip: 192.168.0.144
      access_ip: 192.168.0.144
    worker5:
      ansible_host: 192.168.0.145
      ip: 192.168.0.145
      access_ip: 192.168.0.145
  children:
    kube_control_plane:
      hosts:
        master1:
        master2:
        master3:
    kube_node:
      hosts:
        worker1:
        worker2:
        worker3:
        worker4:
        worker5:
    etcd:
      hosts:
        master1:
        master2:
        master3:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

The command to add more nodes should look like this:
```bash
ansible-playbook -i inventory/myk8s/hosts.yaml  --become --become-user=root scale.yml
```

### Removing node from cluster
To remove node from the cluster we have to run `ansible-playbook` **BEFORE** editing `inventory/myk8s/hosts.yaml`. Let's remove our *worker4* and *worker5* nodes.
```bash
ansible-playbook -i inventory/myk8s/hosts.yaml  --become --become-user=root remove-node.yml --extra-vars "node=worker4,worker5" 
```

Now we can remove them from `inventory/myk8s/hosts.yaml` file.

### Destroying Kubernetes cluster
To destroy existing Kubernetes cluster let's just run `reset.yml`
```bash
ansible-playbook -i inventory/myk8s/hosts.yaml  --become --become-user=root reset.yml
```
and it will completely reset all the nodes.