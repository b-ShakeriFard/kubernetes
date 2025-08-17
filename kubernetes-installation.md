# Kubernetes Installation
This installation method has been tested on CentOS9
and it works well!

### STEP ONE- INSTALL KERNEL HEADERS

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

# STEP THREE - CONFIGURE SYSCTL
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

| Kernel Parameter|	                         Description |
# net.bridge.bridge-nf-call-iptables	      Enables iptables to process bridged IPv4 traffic.
# net.bridge.bridge-nf-call-ip6tables	      Enables iptables to process bridged IPv6 traffic.
# net.ipv4.ip_forward	                      Enables IPv4 packet forwarding.

# Next, apply the changes like so:
sysctl --system

# STEP FOUR - DISABLE SWAP
sudo swapoff -a || true
# Persist across reboots
sed -e '/swap/s/^/#/g' -i /etc/fstab

# this line does the following:
# Find any swap entries in /etc/fstab and comment them out so swap never mounts at startup.
# sed: stream editor, used to search/replace text.
# /swap/ Find any line that contains the word swap (case-sensitive).
# s/^/#/g → "substitute (s) the start of the line (^) with a #", meaning put a # at the beginning (comment it out).
# g → global flag (applies the substitution to all matches in the line, though here it’s redundant because ^ matches only once).
# -i → in-place edit (modifies the file directly).

# STEP FIVE INSTALL CONTAINERD

# Add Docker CE Repository
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# After adding the repository, it’s essential to update the package cache to ensure the latest package information is available:
sudo dnf makecache

# install containerd.io package
sudo dnf install containerd.io

sudo mkdir -p /etc/containerd

# Configure Containerd
cat /etc/containerd/config.toml

# After installing Containerd, the next step is to configure it to ensure optimal performance and 
# compatibility with your environment. The configuration file for Containerd is located at 
# /etc/containerd/config.toml. While the default configuration provides a solid starting point for 
# most environments, we’ll make a small adjustment to enable Systemd Cgroup support, which is 
# essential for proper container management. Let’s proceed with configuring Containerd.

# Run the following command to build out the containerd configuration file:
sudo sh -c "containerd config default > /etc/containerd/config.toml" ; cat /etc/containerd/config.toml

# Ensure CRI plugin is enabled and systemd cgroups are on
# (Handles both containerd 1.x and 2.x config layouts)
sudo sed -ri \
  -e 's/^\s*disabled_plugins\s*=.*/# disabled_plugins = []/' \
  -e 's/(\[plugins\."io\.containerd\.grpc\.v1\.cri"\]\s*)$/\1\nsandbox_image = "registry.k8s.io\/pause:3.10"/' \
  -e 's/(\[plugins\."io\.containerd\.grpc\.v1\.cri"\.containerd\.runtimes\.runc\.options\])/\1\n  SystemdCgroup = true/' \
  -e "s/(\[plugins\.'io\.containerd\.cri\.v1\.runtime'\.containerd\.runtimes\.runc\.options\])/\1\n  SystemdCgroup = true/" \
  /etc/containerd/config.toml
# It is important to make this change in /etc/containerd/config.toml file - changing SystemdCgroup from false to true!

# Now, let's enable, start, and reboot the system!
sudo systemctl enable --now containerd.service

# reboot
sudo systemctl reboot

# check the status
sudo systemctl status containerd.service

# STEP SIX - SET FIREWALL RULES
sudo firewall-cmd --zone=public --permanent --add-port=6443/tcp
sudo firewall-cmd --zone=public --permanent --add-port=2379-2380/tcp
sudo firewall-cmd --zone=public --permanent --add-port=10250/tcp
sudo firewall-cmd --zone=public --permanent --add-port=10251/tcp
sudo firewall-cmd --zone=public --permanent --add-port=10252/tcp
sudo firewall-cmd --zone=public --permanent --add-port=10255/tcp
sudo firewall-cmd --zone=public --permanent --add-port=5473/tcp

# Port(s)	      Description
# 6443	        Kubernetes API server
# 2379-2380	    etcd server client API
# 10250	        Kubelet API
# 10251	        kube-scheduler
# 10252	        kube-controller-manager
# 10255	        Read-only Kubelet API
# 5473	        ClusterControlPlaneConfig API

# let's reload the firewall
sudo firewall-cmd --reload

# STEP SEVEN - INSTALL KUBERNETES COMPONENETS

# Add Kubernetes Repository
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

# Install Kubernetes Packages
dnf makecache; dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# Start and Enable kubelet Service
systemctl enable --now kubelet.service

# Don’t worry about any kubelet errors at this point. Once the worker nodes are 
# successfully joined to the Kubernetes cluster using the provided join command, 
# the kubelet.service on each worker node will automatically activate and start 
# communicating with the control plane. The kubelet is responsible for managing 
# the containers on the node and ensuring that they run according to the specifications 
# provided by the Kubernetes control plane.

# Up until this point of the installation process, we’ve installed and configured 
# Kubernetes components on all nodes. From this point onward, we will focus on the master node.

# STEP EIGHT - INITIALIZING KUBERNETES CONTROL PLANE
sudo kubeadm config images pull

# [config/images] Pulled registry.k8s.io/kube-apiserver:v1.29.3
# [config/images] Pulled registry.k8s.io/kube-controller-manager:v1.29.3
# [config/images] Pulled registry.k8s.io/kube-scheduler:v1.29.3
# [config/images] Pulled registry.k8s.io/kube-proxy:v1.29.3
# [config/images] Pulled registry.k8s.io/coredns/coredns:v1.11.1
# [config/images] Pulled registry.k8s.io/pause:3.9
# [config/images] Pulled registry.k8s.io/etcd:3.5.12-0

# After executing this command, Kubernetes will pull the necessary container images 
# from the default container registry (usually Docker Hub) and store them locally 
# on the machine. This step is typically performed before initializing the Kubernetes cluster 
# to ensure that all required images are available locally and can be used without relying on an 
# external registry during cluster setup.

sudo kubeadm init --pod-network-cidr=10.244.0.0/16

