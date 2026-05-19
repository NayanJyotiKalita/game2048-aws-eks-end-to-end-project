# Before implementing anything in this project, it's very important to install the following tools:

  1. kubectl
  2. eksctl
  3. aws cli
  4. helm

I have given the steps to install the tools in a Linux x86-64 architecture machine, but I have provided the links to their official documentations to suit your system. 

---

# 1. kubectl

### kubectl – A command line tool for working with Kubernetes clusters

Refer Documentation: https://kubernetes.io/docs/tasks/tools/

**Installation Steps:**

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

# 2. eksctl

### eksctl – A command line tool for working with EKS clusters that automates many individual tasks. 

Refer Documentation: https://docs.aws.amazon.com/eks/latest/eksctl/installation.html#_prerequisite

**Installation Steps:**

```
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
```


---

# 3. aws cli

### AWS CLI – A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI in the AWS Command Line Interface User Guide. After installing the AWS CLI, we recommend that you also configure it.

Refer Documentation: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

**Installation Steps:**

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

After this, make sure to configure it by running the `aws configure` command and put the access key details of your AWS user.

---

# 4. helm

### helm - A package manager for Kubernetes that simplifies the deployment and management of applications. It functions similarly to package managers like apt for Ubuntu or Homebrew for macOS, but for the Kubernetes ecosystem.

Refer Documentation: https://helm.sh/docs/intro/install

**Installation Steps:**

```
sudo apt-get install curl gpg apt-transport-https --yes
curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```


























