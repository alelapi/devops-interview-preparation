# OpenSSL commands

## Create new user
1. Generate private key:
```
openssl genrsa -out <private_key_name>.key 2048
```

2. Create Certificate Signing Request:
```
openssl req -new -key <private_key_name>.key -subj "/CN=<user_name>" -out <user_name>.csr
```

3. Create a CSR
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: <user_name>
spec:
  groups:
  - system:authenticated
  request: $REQUEST
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

`$REQUEST` should contains the encoded .csr file:
```
cat <user_name>.csr | base64 | tr -d '\n'
```

4. Approve CSR:
```
kubectl certificate approve <user_name>
```

5. Save client certificate:
```
k get csr <user_name> -o jsonpath='{.status.certificate}' | base64 -d > <user_name>.crt
```

6. Add user to kubeconfig
```
k config set-credentials <user_name> --client-key=<user_name>.key --client-certificate=carlton.crt --embed-certs
```

## Check certificate content
```
openssl x509 -in <certificate_file_name>.crt -text -noout

```