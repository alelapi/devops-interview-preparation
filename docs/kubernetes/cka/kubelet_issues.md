# Kubelet Issues

Search logs:
```
journalctl -u kubelet --no-pager
cat /var/log/syslog | grep kubelet
```

Check status:
```
systemctl status kubelet
```

Restart:
```
systemctl daemon-reload
systemctl restart kubelet
```

Configuration files:
```
/var/lib/kubelet/
/etc/kubernetes/kubelet.conf
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```