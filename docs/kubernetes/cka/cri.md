# CRI

## Install a container runtime interface
### Containerd

Install
```
sudo apt update
sudo apt install -y containerd
```

Setup
```
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Restart
```
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Check if Kubernetes is using containerd
```
ps aux | grep kubelet | grep container-runtime
```

If it's not using it, set it up:
```
sudo systemctl enable --now containerd
```
or:
```
sudo mkdir -p /etc/systemd/system/kubelet.service.d
sudo tee /etc/systemd/system/kubelet.service.d/10-kubeadm.conf <<EOF
[Service]
Environment="KUBELET_EXTRA_ARGS=--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock"
EOF
```

Check which CRI is in use:
```
crictl info
```