# Falco

Falco is an open-source, cloud-native runtime security project that detects unexpected behavior, configuration changes, intrusions, and data theft in real-time. Originally created by Sysdig and now a CNCF (Cloud Native Computing Foundation) graduated project, Falco acts as a security camera for your system, alerting you to potentially malicious activity.

Key features include:
- System call monitoring for process behavior
- Container and Kubernetes monitoring
- Extensible rule language
- Multiple output options (syslog, file, program, HTTP, etc.)
- Low performance overhead

## Basic Configuration

### Configuration File Structure

The main Falco configuration file is typically located at `/etc/falco/falco.yaml`. Here are the key sections:

```yaml
# Rules file(s) or directories
rules_file:
  - /etc/falco/falco_rules.yaml
  - /etc/falco/falco_rules.local.yaml
  - /etc/falco/rules.d

# Where security notifications should be written
output:
  rate: 1
  max_burst: 1000
  enabled: true

# Logging settings
log_stderr: true
log_syslog: true
log_level: info

# Monitoring settings
syscall_event_drops:
  actions:
    - log
    - alert
```

### Output Methods

Configure various output methods in `/etc/falco/falco.yaml`:

#### Standard Output

```yaml
stdout_output:
  enabled: true
```

#### File Output

```yaml
file_output:
  enabled: true
  filename: /var/log/falco.log
  keep_alive: false
```

#### Syslog Output

```yaml
syslog_output:
  enabled: true
```

#### Program Output

```yaml
program_output:
  enabled: true
  program: "curl -d @- -X POST https://example.com/falco-alerts"
```

#### HTTP Output

```yaml
http_output:
  enabled: true
  url: https://alerts.example.com/falco
  user_agent: falco/5.0.0
```

## Working with Falco Rules

### Rule Anatomy

A typical Falco rule consists of:

1. **Rule definition** with a name, description, and priority
2. **Condition** that triggers the rule
3. **Output** format for alerts

Example:

```yaml
- rule: Terminal Shell in Container
  desc: A shell was used as the entrypoint/exec point into a container
  condition: >
    container.id != host and
    proc.name = bash and
    container
  output: >
    Shell executed in a container (user=%user.name %container.info)
  priority: WARNING
  tags: [container, shell]
```

### Creating Custom Rules

Create a new file for custom rules:

```bash
nano /etc/falco/rules.d/custom_rules.yaml
```

Add your rules following this structure:

```yaml
- macro: my_custom_macro
  condition: evt.type=open

- list: my_sensitive_files
  items: [/etc/shadow, /etc/passwd]

- rule: Read Sensitive Files
  desc: Detect attempts to read sensitive files
  condition: >
    open_read and
    fd.name in (my_sensitive_files) and
    not proc.name in (my_trusted_processes)
  output: >
    Sensitive file opened for reading (user=%user.name file=%fd.name)
  priority: WARNING
  tags: [filesystem]
```

After adding rules, restart Falco:

```bash
systemctl restart falco
```

### Testing Rules

Use `falco-tester` to validate your rules:

```bash
falco --validate /etc/falco/falco_rules.yaml

# Test with specific rule file
falco -r /etc/falco/rules.d/custom_rules.yaml -V
```

To test with real events:

```bash
# Run Falco interactively with your rules
falco -r /etc/falco/rules.d/custom_rules.yaml

# In another terminal, perform actions that should trigger rules
cat /etc/shadow
```

## Checking and Analyzing Logs

### Standard Host Logs

#### Systemd Journal

```bash
# View real-time logs
journalctl -u falco -f

# View logs with specific priority
journalctl -u falco -p warning -f

# View logs from a time period
journalctl -u falco --since "2023-04-25 10:00:00" --until "2023-04-25 11:00:00"
```

#### Log Files

If configured to use file output:

```bash
# View the entire log file
cat /var/log/falco.log

# Follow logs in real-time
tail -f /var/log/falco.log

# Filter logs by priority
grep -i "critical\|error" /var/log/falco.log

# Filter logs by rule name
grep "Terminal Shell in Container" /var/log/falco.log
```

#### Syslog

```bash
# Check syslog for Falco events
grep falco /var/log/syslog
```

### Kubernetes Logs

```bash
# List Falco pods
kubectl get pods -n falco

# Follow logs for a specific pod
kubectl logs -n falco falco-abcd1234 -f

# Follow logs for all Falco pods
kubectl logs -n falco -l app=falco -f

# Filter logs by severity
kubectl logs -n falco -l app=falco | grep -i "warning\|error\|critical"
```

### Log Formats and Analysis

#### JSON Output

Configure JSON output for easier parsing:

```yaml
# In falco.yaml
json_output: true
```

Then analyze with tools like `jq`:

```bash
# Extract specific fields
cat /var/log/falco.log | jq 'select(.priority == "CRITICAL")'
cat /var/log/falco.log | jq 'select(.rule == "Terminal Shell in Container")'

# Count alerts by rule
cat /var/log/falco.log | jq -r '.rule' | sort | uniq -c | sort -nr
```

#### Time-Based Analysis

```bash
# Events in the last hour
grep "$(date -d '1 hour ago' +'%Y-%m-%d %H:')" /var/log/falco.log

# Create a timeline of events
cat /var/log/falco.log | jq -r '[.time, .rule, .output] | @tsv' | sort
```

### Alert Management

Configure alert throttling to prevent alert fatigue:

```yaml
# In falco.yaml
outputs:
  rate: 0.03333  # One alert every 30 seconds
  max_burst: 10   # Allow bursts of up to 10 alerts
```

## Advanced Usage

### Custom Field Extraction

Define custom fields for use in rules:

```yaml
# In a custom rules file
- required_engine_version: 11

- source: syscall

- list: trusted_users
  items: ["root", "admin"]

- rule: Custom Field Example
  desc: Using custom fields
  condition: >
    spawned_process and
    not user.name in (trusted_users) and
    proc.args contains "-i"
  output: >
    Process started with interactive flag (user=%user.name command=%proc.cmdline custom_field=%evt.rawtime)
  priority: WARNING
```

### Lua Support

Enhance rules with Lua functions:

```yaml
# Define a Lua function
- macro: is_suspicious_path
  condition: >
    (evt.type=open and fd.name startswith "/tmp/evil" and evt.dir=<)

# Use the function in a rule
- rule: Suspicious File Access
  desc: Accessing potentially malicious files
  condition: is_suspicious_path
  output: Suspicious file accessed (file=%fd.name user=%user.name)
  priority: WARNING
```

### Performance Monitoring

Monitor Falco's performance impact:

```bash
# Check CPU and memory usage
top -p $(pgrep -f falco)

# Check dropped events
grep "Dropped events" /var/log/falco.log

# Use internal metrics (if enabled)
curl http://localhost:8765/metrics
```
