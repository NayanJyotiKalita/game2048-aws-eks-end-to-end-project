# 1. kubectl

### kubectl – A command line tool for working with Kubernetes clusters

**Installation Steps:**

Refer Documentation: https://kubernetes.io/docs/tasks/tools/

Select the system you want to install it into and go ahead, here's the process of installing it in Linux machines:

#### Install kubectl binary with curl on Linux

  1. Download the latest release with the command:
```
# For x86-64 architecture machines
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

To download a specific version, for example, to download version 1.36.0 on Linux x86-64, type:
```
curl -LO https://dl.k8s.io/release/v1.36.0/bin/linux/amd64/kubectl
```

  2. Validate the binary (optional)

Download the kubectl checksum file:
```
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```

Validate the kubectl binary against the checksum file:
```
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
==> kubectl: OK
```

  3. Install kubectl
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

**Note:**
If you do not have root access on the target system, you can still install kubectl to the ~/.local/bin directory:
```
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
```

  4. Test to ensure the version you installed is up-to-date:
```
kubectl version --client
```

---















