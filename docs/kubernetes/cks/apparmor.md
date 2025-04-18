# AppArmor Profile Management

AppArmor is a Linux Security Module that allows system administrators to restrict programs' capabilities with per-program profiles.

## Verifying AppArmor Status

After installation, verify AppArmor is running:

```bash
sudo aa-status
```

## Understanding AppArmor Profiles

AppArmor profiles define what system resources a program can access and what actions it can perform. Profiles can be in one of several modes:

- **enforce**: Restricts the program according to the profile and logs violations
- **complain**: Does not restrict the program, but logs actions that would be prevented in enforce mode
- **disabled**: The profile is loaded but not applied

Profiles are stored in `/etc/apparmor.d/` and have a specific syntax for defining permissions.

## Creating AppArmor Profiles

### Method 1: Using aa-genprof (Recommended for Beginners)

1. Start the profile generation tool:

```bash
sudo aa-genprof /path/to/application
```

2. Run the application to generate typical usage patterns.

3. When done, press 'S' to save the profile.

### Method 2: Creating a Profile Manually

1. Create a new file in `/etc/apparmor.d/` named after your application:

```bash
sudo nano /etc/apparmor.d/my.application
```

2. Add the profile content. Here's a basic example:

```
#include <tunables/global>

profile my.application /path/to/application {
  #include <abstractions/base>
  
  # Allow basic functionality
  /path/to/application mr,
  /usr/lib/** mr,
  /lib/** mr,
  
  # Allow reading of specific files
  /etc/my-app/** r,
  
  # Allow writing to specific directories
  /var/log/my-app/** w,
}
```

### Method 3: Using aa-logprof to Generate from Logs

1. Set an existing profile to complain mode:

```bash
sudo aa-complain /path/to/application
```

2. Run the application to generate logs.

3. Use aa-logprof to analyze logs and update the profile:

```bash
sudo aa-logprof
```

## Loading and Enabling Profiles

### Loading a New Profile

After creating a profile, load it with:

```bash
sudo apparmor_parser -r /etc/apparmor.d/my.application
```

### Setting Profile Mode

Set a profile to enforce mode:

```bash
sudo aa-enforce /path/to/application
```

Set a profile to complain mode:

```bash
sudo aa-complain /path/to/application
```

Disable a profile:

```bash
sudo ln -s /etc/apparmor.d/my.application /etc/apparmor.d/disable/
sudo apparmor_parser -R /etc/apparmor.d/my.application
```

## Managing Profiles

### Listing Profiles

List all profiles and their status:

```bash
sudo aa-status
```

## Using AppArmor with Containers

For Kubernetes, you need to:

1. Create a profile on all worker nodes

2. Load the profile on all nodes:

```bash
sudo apparmor_parser -r /etc/apparmor.d/k8s-myprofile
```

3. Apply the AppArmor profile to your Pod/container using one of two methods:

#### Method 1: Using Annotations (Beta API)

The original beta implementation uses annotations:

```yaml
metadata:
  annotations:
    container.apparmor.security.beta.kubernetes.io/container-name: localhost/k8s-myprofile
```

Where `container-name` is the name of your container, and `k8s-myprofile` is your AppArmor profile name.

#### Method 2: Using securityContext (Preferred)

The newer, more structured approach uses securityContext:

```yaml
# At the container level
spec:
  containers:
  - name: my-container
    securityContext:
      appArmorProfile:
        type: Localhost
        localhostProfile: k8s-myprofile

# OR at the pod level
spec:
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: k8s-myprofile
```

The securityContext approach is recommended for new deployments as it follows Kubernetes conventions for security features and provides better validation.
