Azure Kubernetes Setup Guide (VM → K3s / K8s Cluster)
🧱 PART 1: Create a Virtual Machine (Single Node Setup)
Step 1: Open Azure Portal

Go to:
https://portal.azure.com
Sign in.

Step 2: Create Virtual Machine

Search → Virtual Machines
Click → + Create → Azure Virtual Machine

Step 3: Basic Configuration

Use these settings:

Subscription → Default
Resource Group → Create new: k8s-rg
VM Name → k8s-vm
Region → Closest (e.g. West Europe / North Europe)
Image → Ubuntu Server 22.04 LTS
Size → Standard B2s (2 vCPU, 4GB RAM)
Step 4: Authentication
Authentication type → Password (or SSH key if preferred)
Username → azureuser
Password → Strong password (save it)
Step 5: Networking
Allow inbound port:
✅ SSH (22)

Click → Review + Create → Create

Wait 2–5 minutes.

🔐 PART 2: Connect to VM (MobaXterm / SSH)
Step 1: Get Public IP

Copy the VM Public IP from Azure.

Step 2: Connect using MobaXterm
Session → SSH
Remote Host → Public IP
Username → azureuser
click advanced ssh key > tick use public key > upload the key pem you downloaded on your local computer and click Connect
you will be taken to mobaxterm

to check the hostname type hostnamectl press enter
to change the hostname type sudo hostnamectl set-hostname the new hostname 
example to change the hostname type sudo hostnamectl set-hostname single-k8

update with this cmd sudo apt update -y

Go to browser search for k3s.io. open the k3s documentation on how to install it. copy the code which is curl -sfL https://get.k3s.io | sh -  add this cmd to download it - server --tls-san the vm public ip address the whole cmd curl -sfL https://get.k3s.io | sh -s - server --tls-san 98.71.32.17
this cmd 
curl -sfL https://get.k3s.io | sh -s - server --tls-san 98.71.32.17

curl: This is a tool used to transfer data from a URL. It’s like a web browser without the graphics.

-s (Silent): Tells it not to show progress bars or error messages.

-f (Fail fast): If the website is down, it won't try to download the error page; it just stops.

-L (Location): If the link moves to a new address (a redirect), curl will follow it.

[https://get.k3s.io](https://get.k3s.io): This is the official installation script hosted by Rancher (the creators of K3s).

. The "Pipe" (|)
This vertical bar is called a Pipe. It takes the output from the left side (the text of the script we just downloaded) and "pipes" it directly into the command on the right side. This way, nothing is saved to your hard drive permanently until it's executed.

3. The "Execution" Part (sh -s -)
sh: This is the Shell. It’s the program that reads commands and runs them.

-s -: This tells the shell, "Read the instructions coming from the pipe as if they were a file."

4. The "Configuration" Part (server --tls-san 98.71.32.17)
These are specific instructions passed to the K3s installer:

server: Tells K3s to start as a "Control Plane" (the brain of the cluster), not just a worker node.

--tls-san 98.71.32.17: This is crucial for Cloud VMs. It tells Kubernetes to "bless" your Public IP address. Without this, if you tried to connect to this cluster from your home computer later, the connection would be rejected for security reasons because the cluster wouldn't "recognize" its own public IP.

What will it actually do to your VM?
When you press Enter, the following happens automatically:

Downloads the K3s binary (the actual program).

Configures your VM's networking so the "pods" (containers) can talk to each other.

Sets up a Systemd Service: This means if your VM restarts, Kubernetes will automatically turn itself back on.

Installs kubectl: It gives you the "steering wheel" command-line tool used to manage the cluster.

Generates Keys: It creates the security certificates needed to keep the cluster private.

What this command does
Downloads K3s installer
Installs Kubernetes lightweight cluster
Configures networking
Starts system service automatically
Generates kubeconfig

Verify installation
sudo kubectl get nodes

You should see:

NAME       STATUS   ROLES           AGE
k8s-node   Ready    control-plane   Ready
everything is workingsuccessful

type kubectl and press enter



MULTI-NODES KUBERNETES CLUSTER

STAGE 1: Create 3 VMs in Azure

Create:

️⃣ Control Plane VM
Name: k8s-master
Size: Standard_B2s (2CPU, 4GB RAM)
️⃣ Worker Node 1
Name: k8s-worker-1
Size: Standard_B2s
️⃣ Worker Node 2
Name: k8s-worker-2
Size: Standard_B2s
🔑 IMPORTANT SETTINGS (for ALL VMs)

When creating each VM:

OS → Ubuntu 22.04
Authentication → Password
Open ports:
SSH (22)
Kubernetes later (we’ll handle)
🌐 STAGE 2: Networking (VERY IMPORTANT)

Make sure:

All VMs are in the same Virtual Network (VNet)
Same subnet

👉 This allows them to talk to each other


You ensure all VMs are in the same network

This is very critical for Kubernetes.

When you create the first VM in Microsoft Azure:

Azure creates a Virtual Network (VNet) automatically

For the next VMs, you must:
👉 Select that same VNet and subnet

If you rush and create all at once, you might accidentally:

Put them in different VNets ❌
Cluster will fail ❌
3. Easier naming and tracking

Create like this:

️⃣ First VM

Name: k8s-master

️⃣ Second VM

Name: k8s-worker-1

️⃣ Third VM

Name: k8s-worker-2
🧱 Recommended Step-by-Step Order
Step 1 → Create Master VM
Confirm:
Region
Size (B2s)
VNet name (IMPORTANT)
Step 2 → Create Worker 1
Use:
Same Resource Group
Same Region
Same VNet
Step 3 → Create Worker 2
Same settings again
⚠️ Critical Things to Keep the Same

Across ALL 3 VMs:

✅ Region
✅ Virtual Network (VNet)
✅ Subnet
✅ OS (Ubuntu 22.04)
❌ What NOT to do
Don’t create each VM with “default settings” blindly
Don’t let Azure create new VNet for each VM
Don’t mix regions (e.g., one in Europe, one in US)

connect the master nodes with mobaxterm

Master Node Setup

SSH into master node:

sudo apt update && sudo apt upgrade -y
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab


Enable Kernel Modules
sudo modprobe overlay
sudo modprobe br_netfilter

Kubernetes sysctl config
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system


4) Install Docker or Containerd (Container Runtime)
---------------------------------------------------
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL <https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc>
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo usermod -aG docker ubuntu

sudo systemctl start docker


install containerD
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Enable systemd cgroup
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo sed -i 's|sandbox = "registry.k8s.io/pause:.*"|sandbox = "registry.k8s.io/pause:3.10"|' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd


5) Add the Kubernetes APT Repository
-------------------------------------
sudo apt install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl


Only on Master Node
===================
6) Initialize the Kubernetes Control Plane
------------------------------------------
sudo kubeadm init --pod-network-cidr=10.0.0.0/16

copy this cmd inside the cmd line and run
 mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config


Cat the .kube/config to get the authentication file by using
cat .kube/config


if you run the cmd kubectl get node you will see not ready
you get not ready because the Overlay Network (the "virtual" network) hasn't successfully connected on top of the Underlay Network (the actual Azure physical network).
The Overlay (The "Virtual" Kubernetes Layer) This is the "Overlay." It "lays over" the Azure network.
To solve this 
Install Cilium CNI
8) Install cilium-cli
---------------------
curl -L --remote-name https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz
sudo tar xzvf cilium-linux-amd64.tar.gz -C /usr/local/bin
rm cilium-linux-amd64.tar.gz

 Install Cilium
------------------
cilium install

Wait for 5 minutes


) Check Node Status and Cilium Status
---------------------------------------
cilium status
kubectl get nodes you will see ready


On Worker Nodes

login to server using mobaxterm
create a bash script by using nano workernode-install.sh
open it and paste this configurations
 
#!/bin/bash

# 1) Set Hostname
sudo hostnamectl set-hostname Worker-Node-01

# 2) Update and upgrade the system
sudo apt update && sudo apt upgrade -y

# 3) Disable swap (Required for Kubelet)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# 4) Enable Kernel Modules and Sysctl Settings
sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

# 5) Install Containerd
sudo apt update
sudo apt install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL <https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc>
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
sudo apt install containerd.io -y


# 6) Configure Containerd (The Fix)
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

# Set SystemdCgroup and the correct pause image for v3 config
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo sed -i 's|sandbox = "registry.k8s.io/pause:.*"|sandbox = "registry.k8s.io/pause:3.10"|' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd

# 7) Install Kubeadm, Kubelet
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

exit and save when done

chmod +x workernode-install.sh press enter ll press enter

then type ./workernode-install.sh  and installation will start

after that do same to the 2nd vm 

rename the tabs in mobaxterm

run this cmd kubectl get node
kubectl get node -o wide to give full address

Go up to master node you will see where it is written 
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.1.0.4:6443 --token m5lo7k.h6zvxjyzz25nby8d \
        --discovery-token-ca-cert-hash sha256:6f0f9e45af0b9082930b7bf73cf33741b33939da26f3d6acc5cb0ee2d9c69a01

copy it and paste in workernode 1 and worker node 2 add sudo infront before running and immediately you check kubectl get node -o wide -w you will see them in master nodes

to exit press Ctrl + C then run kubectl get nodes you will see both the master nodes and the 2 worker nodes are ready


install openlens by using chocoinstall openlens using cmd line
open powershell or any cmd line as an administrator
Run the Fresh Install Script
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

HE MOST IMPORTANT STEP (Restart PowerShell)
After the script finishes, you must close your current PowerShell window.
Close the window.
Open a new PowerShell window as Administrator.

Type choco --version

Install OpenLens
Once you see a version number (like 2.x.x), go ahead and install OpenLens:
choco install openlens --params='"/CURRENTUSER"' -y

after installing you will wiher see openlens or freelens both are the same

open it and click on browse clusters in catalog
click on the + sign and select Add from Kubeconfig
you will see this Add Clusters from Kubeconfig
Clusters added here are not merged into the ~/.kube/config
open your masternode terminal and run this cmd ll enter cat .kube/config enter you will see the authentication file copy all and paste in freelens then edit the private ip address go to azure and copy master node public address delete the one there and paste it click on add cluster
when trying to connect it will fail 

go to master node type kubectl get node enter
then paste this cmd what about this kubectl -n kube-system get configmap kubeadm-config -o jsonpath='{.data.ClusterConfiguration}' > kubeadm-config.yaml
open the file by typing nano the file name e.g nano kubeadm-config.yaml
edit the file in the firstline remove {} press enter, in the next line press shift 2 times and write this certSANs: press enter press shift 2 times and type - "private ip address" enter press shift 2 times - "public ip address" like this - "10.1.0.4" enter press shift 3 times - "20.238.19.36" press control x then y and enter


Regenerate the API Server Certificate
Now, tell kubeadm to use that updated config to create a new certificate:

Bash
# Move old certs out of the way
sudo mv /etc/kubernetes/pki/apiserver.crt /etc/kubernetes/pki/apiserver.crt.old
sudo mv /etc/kubernetes/pki/apiserver.key /etc/kubernetes/pki/apiserver.key.old

# Generate new certs using your updated yaml
sudo kubeadm init phase certs apiserver --config kubeadm-config.yaml

delete .kube/config file
rm -rf .kube/config

 Set up kubeconfig
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

cat .kube/config to get the authentication file

go to freelens and delete the one created and paste the new authentication file, edit the private ip address and put the public ip address of the master node and connect

if the connection attempt failed" error is almost certainly a Network Security Group (NSG) or Firewall issue. Even though the certificate is fixed, the Azure virtual machine is likely blocking incoming traffic on port 6443 from the internet.

Think of it like this: the Master node now has its name on the guest list (the certificate), but the front door (the firewall) is still locked.

Step 1: Open Port 6443 in Azure
Since your nodes are in Azure, you need to allow your computer to talk to the Master node.

Log in to the Azure Portal.

Go to your Resource Group and find the Network Security Group (NSG) attached to your Master-Nodes VM.

Click on Inbound security rules.

Add a new rule:

Source: Any (or My IP address for better security).

Source port ranges: *

Destination: Any

Service: Custom

Destination port ranges: 6443

Protocol: TCP

Action: Allow

Priority: 100 (or anything lower than the default "DenyAll").

Name: AllowK8sAPI

then go back to freelens and connect it will now connect



The "Quick Command" for DevOps Engineers
If you have the Azure CLI installed, you can run this command to list every single resource across all regions in one go:  
PowerShell
az resource list --output table
If this command returns a blank table, your subscription is officially empty.
