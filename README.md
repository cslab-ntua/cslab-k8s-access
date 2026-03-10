# CSLAB Kubernetes Access Guide and Usage Examples

## 1. Introduction

This guide explains how to connect to the CSLAB  infrastructure through VPN, install the required tools, set up your local working environment, and run Kubernetes (k8s) tasks.

[Kubernetes (K8s)](https://kubernetes.io/) is an open-source platform for managing containerized applications at scale. It helps automate deployment, management, and scaling of applications.

You will also receive an email containing two configuration files and a username for accessing the infrastructure. In the instructions below, whenever you see **`<username>`**, replace it with the username you received by email.

The course material is available in the following repository:

https://github.com/ikons/cslab-k8s-access

You can clone it locally with:

```bash
cd ~
git clone https://github.com/cslab-ntua/cslab-k8s-access
```

This command copies the entire repository to your local machine. Since the repository may be updated regularly, make sure you keep it up to date by running:

```bash
cd cslab-k8s-access
git pull
```

## 2. Installing OpenVPN Client, kubectl, and k9s

To connect to the infrastructure, first install the [OpenVPN client](https://openvpn.net/community-downloads/).

After installing it, import the `.ovpn` file you received by email and connect to the VPN.

### Installing `kubectl`

`kubectl` is the command-line tool for managing Kubernetes clusters. Install it on a Linux machine or WSL using the commands below:

```bash
# Install the basic packages required for HTTPS repositories
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

# Download and store the public key for the Kubernetes repository
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Set the correct permissions on the key file
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes repository to the apt sources list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Set the correct permissions on the repository file
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list

# Update the apt package index
sudo apt-get update

# Install kubectl
sudo apt-get install -y kubectl

# Create the ~/.kube directory where the config file will be stored
mkdir ~/.kube
```

Next, place the `config` file you received by email in `~/.kube/config` so that `kubectl` can connect to the Kubernetes infrastructure.

To do this, copy the `config` file from the location where you originally downloaded it on your host machine (for example, Windows) into the `~/.kube` directory inside Linux/WSL.

Assume that you downloaded the `config` file into your Windows user's `Downloads` folder.

Run the following commands inside WSL Linux, replacing **`<username>`** with your Windows username. For example, in my case the path is `/mnt/c/Users/ikons/Downloads/config`.

```bash
# Go to your home directory
cd

# Create the .kube directory if it does not already exist
mkdir .kube

# Copy the config file from the Windows file system into WSL
cp /mnt/c/Users/<username>/Downloads/config ~/.kube/config
```

You can also do this through Windows Explorer by browsing to the Linux folder.

### Installing `k9s`

`k9s` is a terminal-based tool for monitoring and managing Kubernetes clusters. Install it with:

```bash
wget https://github.com/derailed/k9s/releases/download/v0.40.10/k9s_linux_amd64.deb
sudo dpkg -i k9s_linux_amd64.deb
echo "export KUBE_EDITOR=nano" >> ~/.bashrc
```

`k9s` uses the same configuration file: `~/.kube/config`.

### Monitoring Workloads with `k9s`

To monitor the workload you just submitted, run:

```bash
k9s
```

### Useful `k9s` Commands

**Show pods:**

```bash
:pods
```

**View the logs of a pod:**

```bash
l
```

**Inspect the status/details of a pod:**

```bash
d
```

## 3. Writing a Manifest and Running It with `kubectl`

A **manifest** is a YAML file that describes Kubernetes resources. You can create and apply these files using `kubectl apply`.

Go to the example directory:

```bash
cd ~/cloud-uth/code/03_manifest
```

### Example Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

To create the Pod shown above:

```bash
kubectl apply -f nginx-pod.yaml
```

To verify that the Pod was created successfully:

```bash
kubectl get pod nginx -o wide
```

You should see output similar to this:

```text
NAME    READY   STATUS    RESTARTS   AGE     IP               NODE              NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          7h34m   10.233.101.168   source-code-pc6   <none>           <none>
```

You can use the Pod IP address to open it in your browser and view the default nginx welcome page:

```text
http://10.233.101.168
```

## 4. Cleanup

Once you have confirmed that the nginx Pod is running correctly and serving the default nginx page, you can remove it from the cluster with:

```bash
# Delete the Pod
kubectl delete -f nginx-pod.yaml
```