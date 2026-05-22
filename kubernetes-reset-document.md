
# Kubernetes Full Cluster Reset Script (Master & Worker Nodes)

## 📌 Purpose

This script completely removes Kubernetes components, container runtime, networking configurations, and related system artifacts from both master and worker nodes. It is used for full cluster reinitialization.

---

## ⚠️ Warning

* This is a **destructive operation**
* It will permanently delete Kubernetes and Docker/container data
* Run on **ALL nodes (master + workers)** carefully

---

## 🧾 Full Cleanup Script 

```bash
echo "Cluster Will Be Deleted In 0"
sleep 1

sudo rm -rf /etc/cni/net.d/
sudo rm -rf /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo rm -rf /etc/kubernetes/kubelet.conf \
            /etc/kubernetes/pki/ca.crt \
            /etc/kubernetes/bootstrap-kubelet.conf

sudo netstat -tupln | grep -E "10250|10256|10257|10259|6443" | \
awk '{print $7}' | cut -d'/' -f1 | grep -E '^[0-9]+$' | \
xargs -r sudo kill -9

sudo rm -rf /etc/apt/sources.list.d/kubernetes.list

sudo apt-mark unhold kubelet kubeadm kubectl

sudo apt-get purge -y \
docker docker-ce docker-ce-cli containerd.io \
docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras

sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd

sudo kubeadm reset --force

sudo systemctl disable --now kubelet

sudo apt-get purge -y kubelet kubeadm kubectl

sudo apt autoremove -y
sudo apt-get clean -y
sudo apt-get update -y

rm -rf /home/kube/cluster_initialized.log
rm -rf /home/kube/pod_network_setup.log
rm -rf /home/kube/node_joined.log
```

---

## ⚙️ How to Use This Script

### 1. Create the script file

```bash
nano cluster_reset.sh
```

### 2. Paste the above script into the file

Save and exit:

* Press `CTRL + X`
* Press `Y`
* Press `Enter`

---

### 3. Give execution permission

```bash
chmod 700 cluster_reset.sh
```

---

### 4. Execute the script

```bash
./cluster_reset.sh
```

---

## 🧠 Summary

This script performs:

* Kubernetes reset (`kubeadm reset`)
* Removal of kubelet/kubeadm/kubectl
* Docker & containerd cleanup
* Network/CNI cleanup
* System package cleanup
* Process termination on Kubernetes ports
* Log file cleanup

---

