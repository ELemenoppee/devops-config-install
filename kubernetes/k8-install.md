# 🌻 Set Up Kubernetes: A Step-by-Step Guide 🌻

## Description

Are you looking to deploy a Kubernetes cluster on AWS EC2 but not sure where to start? This comprehensive guide will walk you through the entire process, from setting up your AWS environment to deploying your first application on your new Kubernetes cluster. By following this procedure, you'll gain a solid understanding of how to leverage Virtual Machines for your Kubernetes infrastructure, ensuring a scalable and reliable deployment.

## Steps:-

**Step 1** — Setup hostname on each node

**Step 2** — Disable swap and add kernel parameters

**Step 3** — Install containerd runtime

**Step 4** — Add apt repository for kubernetes

**Step 5** — Install kubectl, kubeadm, kubelet

**Step 6** — Install kubernetes cluster

**Step 7** — Join worker nodes to the cluster

**Step 8** — Install calico network plugin

## Now, let's get started:-

**Step 1** — Setup hostname on each node

Login to master and worker node then set hostname using hostnamectl command

```
$ sudo hostnamectl set-hostname "k8smaster"
$ exec bash 
```

On the work worker nodes, run
```
$ sudo hostnamectl set-hostname "k8sworker1"   // 1st worker node
$ sudo hostnamectl set-hostname "k8sworker2"   // 2nd worker node
$ exec bash
```

Add the following entries in /etc/hosts file on each node
```
192.168.1.173   k8smaster
192.168.1.174   k8sworker1
192.168.1.175   k8sworker2
```

**Step 2** — Disable swap and add kernel parameters

Execute beneath swapoff and sed command to disable swap. Make sure to run the following commands on all the nodes.
```
$ sudo swapoff -a
$ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Load the following kernel modules on all the nodes
```
$ sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```

Set the following Kernel parameters for Kubernetes, run beneath tee command
```
$ sudo tee /etc/sysctl.d/kubernetes.conf <<EOT
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOT
```

Reload the above changes, run
```
$ sudo sysctl –system
```

**Step 3** — Install containerd runtime

In this guide, we are using containerd runtime for our Kubernetes cluster. So, to install containerd, first install its dependencies.

```
$ sudo apt install -y curl software-properties-common apt-transport-https ca-certificates
```

Enable docker repository
```
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Now, run following apt command to install containerd
```
$ sudo apt update
$ sudo apt install -y containerd.io
```

Configure containerd so that it starts using systemd as cgroup.
```
$ containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
$ sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

Restart and enable containerd service
```
$ sudo systemctl restart containerd
$ sudo systemctl enable containerd
```

**Step 4** — Add apt repository for kubernetes

Kubernetes package is not available in the default Ubuntu 22.04 package repositories. So we need to add Kubernetes repository. run following command to download public signing key
```
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Next, run following echo command to add Kubernetes apt repository.
```
$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

**Note:** At the time of writing this guide, Kubernetes v1.28 was available, replace this version with new higher version if available.

**Step 5** — Install kubectl, kubeadm, kubelet

Post adding the repositories, install Kubernetes components like kubectl, kubelet and Kubeadm utility on all the nodes. Execute following set of commands
```
$ sudo apt update
$ sudo apt install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

**Step 6** — Install kubernetes cluster

Now, we are all set to initialize Kubernetes cluster. Run the following Kubeadm command on the master node only
```
$ sudo kubeadm init --pod-network-cidr=10.1.0.0/16
```

After the initialization is complete, you will see a message with instructions on how to join worker nodes to the cluster. Make a note of the kubeadm join command for future reference.

So, to start interacting with cluster, run following commands on the master node,
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

next, try to run following kubectl commands to view cluster and node status
```
$ kubectl cluster-info
$ kubectl get nodes
```

**Step 7** — Join worker nodes to the cluster

On each worker node, use the kubeadm join command you noted down earlier after initializing the master node on step 6. It should look something like this:
```
$ kubeadm join 192.168.1.173:6443 --token 6yhkqu.821gnlcer702ts8w \
        --discovery-token-ca-cert-hash sha256:7940e110a21c33f5f102ad46475be88e6e3090dc57c2a8a1b3b5a43a55b1eb1f
```

**Step 8** — Install calico network plugin

A network plugin is required to enable communication between pods in the cluster. Run following kubectl command to install Calico network plugin from the master node,
```
$ kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```

## Final Note

If you find this repository useful for learning, please give it a star on GitHub. Thank you!

**Authored by:** [ELemenoppee](https://github.com/ELemenoppee)
