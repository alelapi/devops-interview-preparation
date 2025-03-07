# ETDC

Backup ETCD database:

1. Get certicates paths:
```
kubectl get pod etcd-controlplane -n kube-system -o yaml
```

2. Create snapshot
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> snapshot save <backup-file-location>
```

3. Restore snapshot
```
ETCDCTL_API=3 etcdctl --data-dir <data-dir-location> snapshot restore <backup-file-location>
```