# Kubernetes Installation
This installation method has been tested on CentOS9
and it works well!

### STEP ONE - INSTALL KERNEL HEADERS

kernel-devel is the package that contains header files and source code needed to <br>
build kernel modules. It’s a way to make sure your kernel headers match your <br>
currently running kernel so that module compilation succeeds without <br>
version mismatch errors. <br>
<br>
`sudo dnf install kernel-devel-$(uname -r)`

<hr>

### STEP TWO - ADD KERNEL MODULES
#### Let's load bridge netfilter module <br>
This lets iptables inspect and filter traffic passing through <br>
Linux bridges (which Kubernetes CNIs use to connect pods). <br>
<br>
`sudo modprobe br_netfilter`

#### Let's load IP Virtual Server core modules <br>
IPVS is a kernel-level load balancer. <br>
<br>
`sudo modprobe iv_vs`

#### Let's load the Round-Robin scheduler for IPVS.
Round Robin distributes connections evenly across all backend pods. <br>
<br>
`sudo modprobe ip_vs_rr`

#### Let's load the Weighted Round-Robin scheduler for IPVS.
Same as RR, but lets you give more traffic to certain pods (weights). <br>
<br>
`sudo modprobe ip_vs_wrr`

#### Let's load the Source Hashing scheduler for IPVS.
Uses the client’s IP to always map them to the same backend pod (good for session stickiness). <br>
<br>
`sudo modprobe ip_vs_sh`

#### Let's load the overlay filesystem module.
OverlayFS lets container runtimes (like containerd, Docker, CRI-O) stack multiple <br>
filesystem layers — crucial for container images. <br>
<br>

`sudo modprobe overlay`

#### Cool - We are now done with loading modules.
<hr>
Next, we will now add these to our kubernetes.conf file!<br>
<br>

```bash
cat > /etc/modules-load.d/kubernetes.conf << EOF
br_netfilter
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
overlay
EOF
```

## STEP THREE - CONFIGURE SYSCTL
To set specific sysctl settings (on each node) that Kubernetes relies on, you can <br>
update the system’s kernel parameters. These settings ensure optimal performance and <br>
compatibility for Kubernetes. Here’s how you can configure the necessary sysctl settings: <br>

```bash
cat > /etc/sysctl.d/kubernetes.conf << EOF
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

| Kernel Parameter | Description |
| -----------------|-------------|
| net.bridge.bridge-nf-call-iptables | Enables iptables to process bridged IPv4 traffic |
| net.bridge.bridge-nf-call-ip6tables | Enables iptables to process bridged IPv6 traffic |
| net.ipv4.ip_forward	| Enables IPv4 packet forwarding |

<br>

Next, apply the changes like so: <br>
`sysctl --system`

<hr>

## STEP FOUR - DISABLE SWAP

We need to do this! <br>
`sudo swapoff -a || true`
<br>
Persist across reboots <br>
`sed -e '/swap/s/^/#/g' -i /etc/fstab`

#### This line does the following:
- Find any swap entries in /etc/fstab and comment them out so swap never mounts at startup.
- sed: stream editor, used to search/replace text.
- /swap/ Find any line that contains the word swap (case-sensitive).
- s/^/#/g → "substitute (s) the start of the line (^) with a #", meaning put a # at the beginning (comment it out).
- g → global flag (applies the substitution to all matches in the line, though here it’s redundant because ^ matches only once).
- -i → in-place edit (modifies the file directly).


## STEP FIVE - INSTALL CONTAINERD

Add Docker CE Repository <br>
`sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

After adding the repository, it’s essential to update the package cache to ensure the latest package information is available:
`sudo dnf makecache`

### install containerd.io package
`sudo dnf install containerd.io`

Now, let's make some preps! <br>
`sudo mkdir -p /etc/containerd`

### Configure Containerd
`cat /etc/containerd/config.toml`

<br>

After installing Containerd, the next step is to configure it to ensure optimal performance and <br>
compatibility with your environment. The configuration file for Containerd is located at <br>
`/etc/containerd/config.toml`. While the default configuration provides a solid starting point for <br>
most environments, we’ll make a small adjustment to enable Systemd Cgroup support, which is <br>
essential for proper container management. Let’s proceed with configuring Containerd. <br>

<br>

Run the following command to build out the containerd configuration file: <br>
`sudo sh -c "containerd config default > /etc/containerd/config.toml" ; cat /etc/containerd/config.toml`

<br>

### Ensure CRI plugin is enabled and systemd cgroups are on <br>
(Handles both containerd 1.x and 2.x config layouts) <br>
```bash
sudo sed -ri \
  -e 's/^\s*disabled_plugins\s*=.*/# disabled_plugins = []/' \
  -e 's/(\[plugins\."io\.containerd\.grpc\.v1\.cri"\]\s*)$/\1\nsandbox_image = "registry.k8s.io\/pause:3.10"/' \
  -e 's/(\[plugins\."io\.containerd\.grpc\.v1\.cri"\.containerd\.runtimes\.runc\.options\])/\1\n  SystemdCgroup = true/' \
  -e "s/(\[plugins\.'io\.containerd\.cri\.v1\.runtime'\.containerd\.runtimes\.runc\.options\])/\1\n  SystemdCgroup = true/" \
  /etc/containerd/config.toml
```

[!Tip] ### It is important to make this change in `/etc/containerd/config.toml` file - changing SystemdCgroup from false to true!

#### Now, let's enable, start, and reboot the system!
`sudo systemctl enable --now containerd.service`

<br>

#### reboot
`sudo systemctl reboot`

#### check the status
`sudo systemctl status containerd.service`

## STEP SIX - SET FIREWALL RULES

self-explanatory! <br>

```bash
sudo firewall-cmd --zone=public --permanent --add-port=6443/tcp
sudo firewall-cmd --zone=public --permanent --add-port=2379-2380/tcp
sudo firewall-cmd --zone=public --permanent --add-port=10250/tcp
sudo firewall-cmd --zone=public --permanent --add-port=10251/tcp
sudo firewall-cmd --zone=public --permanent --add-port=10252/tcp
sudo firewall-cmd --zone=public --permanent --add-port=10255/tcp
sudo firewall-cmd --zone=public --permanent --add-port=5473/tcp
```

| Port(s) | Description |
|---------|-------------|
| 6443 | Kubernetes API server |
| 2379-2380	| etcd server client API |
| 10250	| kubelet API |
| 10251	| kube-scheduler |
| 10252	| kube-controller-manager |
| 10255	| Read-only Kubelet API |
| 5473 | ClusterControlPlaneConfig API |

#### Let's reload the firewall!
`sudo firewall-cmd --reload`


## STEP SEVEN - INSTALL KUBERNETES COMPONENETS

### Add Kubernetes Repository

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```

#### Install Kubernetes Packages
`dnf makecache; dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`

#### Start and Enable kubelet Service
`systemctl enable --now kubelet.service`

<hr>

Don’t worry about any kubelet errors at this point. Once the worker nodes are <br>
successfully joined to the Kubernetes cluster using the provided join command, <br>
the kubelet.service on each worker node will automatically activate and start <br>
communicating with the control plane. The kubelet is responsible for managing <br>
the containers on the node and ensuring that they run according to the specifications <br>
provided by the Kubernetes control plane. <br>

<hr>

Up until this point of the installation process, we’ve installed and configured <br>
Kubernetes components on all nodes. From this point on-ward, we will focus on the master node. <br>

## STEP EIGHT - INITIALIZING KUBERNETES CONTROL PLANE

Take a guess! We need some images:
`sudo kubeadm config images pull`

```bash
[config/images] Pulled registry.k8s.io/kube-apiserver:v1.29.3
[config/images] Pulled registry.k8s.io/kube-controller-manager:v1.29.3
[config/images] Pulled registry.k8s.io/kube-scheduler:v1.29.3
[config/images] Pulled registry.k8s.io/kube-proxy:v1.29.3
[config/images] Pulled registry.k8s.io/coredns/coredns:v1.11.1
[config/images] Pulled registry.k8s.io/pause:3.9
[config/images] Pulled registry.k8s.io/etcd:3.5.12-0
```

After executing this command, Kubernetes will pull the necessary container images <br>
from the default container registry (usually Docker Hub) and store them locally <br>
on the machine. This step is typically performed before initializing the Kubernetes cluster <br>
to ensure that all required images are available locally and can be used without relying on an <br>
external registry during cluster setup. <br>

`sudo kubeadm init --pod-network-cidr=10.244.0.0/16`

```bash
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

kubeadm join 192.168.1.26:6789 --token y8cow4.jib2syhyrb0bh1dt \
	--discovery-token-ca-cert-hash sha256:cb67fddec41469cf1f495db34008ae1a41d3f24ce418b46d5aefb262a1721f43
```

### Set Up kubeconfig File
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Deploy Pod Network
`kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml`

For this example, we will use tigera-operator. But this is only one of many solutions. <br>
Once you get the content of the file (which is used to create a kubernetes object within it's <br>
own namespace) you will have the privillege to read (receive) these messages:

```bash
namespace/tigera-operator created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created
serviceaccount/tigera-operator created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
deployment.apps/tigera-operator created
```

Now, you are the master of your own single-node kubernetes cluster! <br>
So use your powers wisely!

<hr>

### But...
We are not done yet. Do this: <br>
`wget https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml`

<br>
Adjust the CIDR <br>
`sed -i 's/cidr: 192\.168\.0\.0\/16/cidr: 10.244.0.0\/16/g' custom-resources.yaml`

<br>
### Finally... <br>
Create the Calico custom resource:

`kubectl create -f custom-resources.yaml`

#Boom!
Now You may enjoy your single-node application platform, managed by kubernetes! <br>

You're welcome! <br>

Also, thank this guy:

Reference: https://infotechys.com/install-a-kubernetes-cluster-on-rhel-9/
