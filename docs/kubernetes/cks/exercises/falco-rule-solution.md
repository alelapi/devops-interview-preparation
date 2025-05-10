# Falco Rule to Detect Container Access to /dev/mem

## Problem Statement

Create a Falco rule to detect which container is accessing the `/dev/mem` folder. The rule output should include Kubernetes namespace name and pod/container ID information.

## Solution

Falco is an open-source cloud-native runtime security tool that can detect anomalous behavior in your containers and applications. For this task, we need to create a rule that specifically monitors for container access to the `/dev/mem` device, which could indicate a privileged container attempting to access physical memory directly (a potential security concern).

### Rule Implementation

```yaml
- rule: Container Accessing /dev/mem
  desc: Detect attempts by containers to access /dev/mem device
  condition: >
    open_read and 
    container and 
    fd.name = "/dev/mem" and 
    not falco_privileged_containers
  output: >
    Container accessed /dev/mem device 
    (user=%user.name user_uid=%user.uid 
    command=%proc.cmdline %container.info 
    pod=%k8s.pod.name ns=%k8s.ns.name container_id=%container.id image=%container.image.repository)
  priority: WARNING
  tags: [container, access, memory, dev, k8s]
```

### Rule Explanation

Let's break down the components of this rule:

#### Rule Metadata
- **rule**: Name of the rule - "Container Accessing /dev/mem"
- **desc**: Brief description of what the rule detects
- **priority**: Set to WARNING as this is potentially suspicious activity 
- **tags**: Keywords for organizing and categorizing the rule

#### Condition
The condition defines when the rule should trigger:

```
open_read and container and fd.name = "/dev/mem" and not falco_privileged_containers
```

This condition will match when:
- An `open_read` syscall occurs (a file is opened for reading)
- The activity happens within a container
- The file being accessed is specifically `/dev/mem`
- The container is not already in the list of known privileged containers (to reduce false positives)

#### Output
The output specifies what information to include in the alert:

```
Container accessed /dev/mem device (user=%user.name user_uid=%user.uid command=%proc.cmdline %container.info pod=%k8s.pod.name ns=%k8s.ns.name container_id=%container.id image=%container.image.repository)
```

This will output:
- The username and UID of the process
- The command that was run
- Container information
- **Pod name** (as required)
- **Kubernetes namespace** (as required)
- **Container ID** (as required)
- Container image information

### Testing the Rule

1. Save the rule to a file (e.g., `dev-mem-access.yaml`)
2. Deploy it to your Falco configuration:
   ```bash
   sudo cp dev-mem-access.yaml /etc/falco/rules.d/
   ```
3. Restart Falco:
   ```bash
   sudo systemctl restart falco
   ```

4. To test the rule, you can run a privileged container that attempts to access `/dev/mem`:
   ```bash
   kubectl run test-pod --image=alpine --overrides='{"spec": {"containers": [{"name": "test-pod", "image": "alpine", "command": ["sh", "-c", "sleep 3 && cat /dev/mem 2>/dev/null || echo 'Access attempted'"], "securityContext": {"privileged": true}}]}}'
   ```

### Example Alert Output

When the rule is triggered, you should see an alert in the Falco logs that looks like:

```
Warning Container Accessing /dev/mem: Container accessed /dev/mem device (user=root user_uid=0 command=cat /dev/mem container=test-pod pod=test-pod-6f7d9c5b9-xv7tj ns=default container_id=e8913b725fd3 image=alpine)
```

## Additional Considerations

1. This rule might generate false positives in environments where legitimate access to `/dev/mem` is needed.

2. Consider adding a macro if you have specific containers that legitimately need to access `/dev/mem`:
   ```yaml
   - macro: authorized_mem_access_containers
     condition: (container.name in (authorized-container-1, authorized-container-2))
   ```

   Then modify the rule condition:
   ```yaml
   condition: open_read and container and fd.name = "/dev/mem" and not falco_privileged_containers and not authorized_mem_access_containers
   ```

3. For production environments, you might want to increase the priority to ERROR if this activity is strictly prohibited in your security policies.

## Conclusion

This Falco rule effectively monitors and alerts on container access to the `/dev/mem` device with the required Kubernetes namespace and container identification information included in the output.
