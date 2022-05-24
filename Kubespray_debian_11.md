## Step 1: Prepare your servers:

```
sudo apt update
sudo apt upgrade
```


## Step 2: Prepare your ansible master-machine.

1) Login with your user (not root) and add sudo:

```
usermod -aG sudo pi
```

2) Change your hosts file

```
sudo nano /etc/hosts
```
```
51.250.111.74   master0
84.252.142.192  worker1
84.252.136.96   worker2
```

3) Create and add keys to remote hosts:

```
ssh-keygen
ssh-copy-id pin@master0
ssh-copy-id pin@worker1
ssh-copy-id pin@worker2
```

4) Clone kubespray and install dependencies:

```
cd ~
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
sudo apt install python3-pip
sudo pip3 install -r requirements.txt
ansible --version
```

5) Create your directory of inventory:

```
cp -rfp inventory/sample inventory/mycluster
```

6) Change servers in the inventory:

```
sudo nano inventory/mycluster/inventory.ini
```

Example: 

```
# ## Configure 'ip' variable to bind kubernetes services on a
# ## different ip than the default iface
# ## We should set etcd_member_name for etcd cluster. The node that is not a etcd member do not need to set the value, or can set the empty string value.

[all]
master0   ansible_host=51.250.111.74       ansible_user=pin
worker1   ansible_host=84.252.142.192      ansible_user=pin
worker2   ansible_host=84.252.136.96       ansible_user=pin

# ## configure a bastion host if your nodes are not directly reachable
# [bastion]
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube_control_plane]
master0

[etcd]
master0

[kube_node]
worker1
worker2

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

7) Change settings of the cluster:

```
sudo nano inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
sudo nano inventory/mycluster/group_vars/all/all.yml
```

## Step 3: Create your cluster:

```
ansible-playbook -i inventory/mycluster/inventory.ini --become --user=pin --become-user=root cluster.yml
```

## Step 4: Create your cluster on master:

```
sudo kubectl get nodes
```
