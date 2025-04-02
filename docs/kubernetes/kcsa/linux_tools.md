# AppArmor

AppArmor (Application Armor) is a Linux security module that provides mandatory access control (MAC) for processes. It restricts programs' capabilities by enforcing security policies that limit what actions applications can perform.

```
# Check AppArmor status
sudo aa-status

# List all profiles and their modes
sudo apparmor_status

# Get version information
sudo apparmor_parser -V

# Load a profile
sudo apparmor_parser -r /etc/apparmor.d/profile_name

# Set a profile to enforce mode
sudo aa-enforce /path/to/binary

# Set a profile to complain mode
sudo aa-complain /path/to/binary

# Check profile syntax
sudo apparmor_parser -p /etc/apparmor.d/profile_name

```

## Kubernetes AppArmor Integration
Kubernetes supports AppArmor profiles through annotations on pods. This allows you to apply different security profiles to different pods based on their specific security requirements.

### Implementation Details

- **Pod Annotations**: AppArmor profiles are specified using annotations on the pod specification
- **Node Requirements**: AppArmor must be installed on each worker node
- **Profile Loading**: Profiles must be loaded on each node before pods can use them

```
apiVersion: v1
kind: Pod
metadata:
  name: secure-nginx
  annotations:
    container.apparmor.security.beta.kubernetes.io/nginx: runtime/default
spec:
  containers:
  - name: nginx
    image: nginx
```