# Web Security Scanner

## Core Concepts

Web Security Scanner automatically scans web applications for common vulnerabilities. Identifies security issues before attackers exploit them.

**Key Principle**: Automated vulnerability scanning for web apps; complements manual security reviews.

## What It Scans For

**OWASP Top 10**:

- Cross-site scripting (XSS)
- Flash injection
- Mixed content (HTTP in HTTPS)
- Outdated/insecure libraries
- Clear text password submission

**Other Vulnerabilities**:

- Invalid content types
- Invalid headers
- XSS via AngularJS templates
- Insecure JavaScript libraries

**Does NOT scan for**:

- SQL injection (complex, risk of data corruption)
- Authentication/authorization flaws
- Business logic vulnerabilities
- Zero-day vulnerabilities
- Server misconfigurations (use Cloud Asset Inventory)

## When to Use

### ✅ Use When

- Need automated security testing
- Continuous vulnerability scanning
- Pre-production testing
- Compliance requirements (demonstrate scanning)
- Part of CI/CD pipeline
- Public-facing web applications

### ❌ Don't Use When

- Production databases (no SQLi scanning)
- APIs without web UI → Use manual testing
- Need comprehensive penetration testing → Hire experts
- Real-time protection needed → Cloud Armor

## Scan Types

### Managed Scans (Recommended)

**Automatic**:

- App Engine, Cloud Run, GKE with Ingress
- Automatic discovery
- Scheduled scans

**Benefits**: No configuration, continuous monitoring

### Custom Scans

**Manual setup**:

- Specify target URL
- Configure authentication
- Set scan schedule

**Use for**: Compute Engine, external hosting, custom setups

## Authentication

**Supported**:

- **Google Account**: For protected App Engine apps
- **Custom login**: Record login sequence

**Process**:

1. Configure authentication in scanner
2. Scanner logs in before scanning
3. Scans authenticated pages

**Limitation**: Basic auth flows only (no complex 2FA/MFA)

## Scan Frequency

**Options**:

- On-demand (manual trigger)
- Daily
- Weekly
- Monthly

**Recommendation**: Weekly for production, daily for high-risk apps

**Considerations**: Scanner generates traffic (test on staging first)

## Results and Reporting

### Finding Severity

- **High**: Critical vulnerabilities, immediate action
- **Medium**: Important issues, plan remediation
- **Low**: Minor issues, good practice

**False Positives**: Possible, verify findings

### Integration

**Security Command Center**: Centralized findings
**Cloud Monitoring**: Alerts on new vulnerabilities
**Cloud Logging**: Scan execution logs

## Architecture Patterns

### CI/CD Integration

```
Code commit → Build → Deploy to staging → Web Security Scanner → 
If pass → Deploy to production
If fail → Alert developers
```

### Multi-Environment Scanning

```
Dev environment: Weekly scans
Staging: Daily scans (pre-production validation)
Production: Monthly scans (compliance)
```

### Defense in Depth

```
Layer 1: Secure coding practices
Layer 2: Web Security Scanner (find issues)
Layer 3: Cloud Armor (runtime protection)
Layer 4: Security reviews (manual)
```

## Limitations

### Coverage Limitations

- No SQLi testing (safety concerns)
- No authenticated session testing (limited)
- No API testing (web UI only)
- No file upload testing
- No business logic testing

### Technical Limitations

- Max 100 starting URLs
- Max scan duration: 24 hours
- Rate limited (won't overwhelm site)
- Crawls only linked pages (no fuzzing)

### False Negatives

- May miss vulnerabilities
- Not replacement for penetration testing
- No zero-day detection
- Limited coverage of complex apps

## Cost

**Pricing**: Free (no charge for Web Security Scanner)

**Related costs**: Compute resources being scanned

## Web Security Scanner vs Alternatives

| Need | Solution |
|------|----------|
| Automated scanning | Web Security Scanner |
| Comprehensive pentest | Third-party security firm |
| Runtime protection | Cloud Armor |
| Code analysis | Static analysis tools |
| API security testing | OWASP ZAP, Burp Suite |
| Container scanning | Binary Authorization, Artifact Analysis |

## Best Practices

### Scan Staging First

**Pattern**: Always test scanner on non-production

**Reason**: Verify behavior, avoid production impact

### Review Findings Promptly

**Process**:

1. Scan completes
2. Review findings same day
3. Prioritize by severity
4. Fix high/medium issues
5. Re-scan to verify

### Combine with Manual Testing

**Strategy**: Scanner + manual reviews + penetration testing

**Frequency**: Scanner (continuous), manual (quarterly), pentest (annually)

### Integrate into CI/CD

**Automation**: Scan on every staging deployment

**Gates**: Block production deployment if high-severity findings

## Remediation Guidance

**Provided**:

- Vulnerability description
- Affected URL
- Evidence (sample request/response)
- Remediation steps

**Example - XSS**:

- Finding: Reflected XSS on search page
- Evidence: `<script>alert(1)</script>` reflected
- Remediation: Sanitize user input, encode output

## Common Findings

### Mixed Content

**Issue**: HTTPS page loading HTTP resources

**Fix**: Use HTTPS for all resources or protocol-relative URLs

### XSS

**Issue**: User input reflected without sanitization

**Fix**: Input validation, output encoding, CSP headers

### Outdated Libraries

**Issue**: Using vulnerable JavaScript libraries

**Fix**: Update to latest secure versions

### Clear Text Passwords

**Issue**: Login form submits over HTTP

**Fix**: Use HTTPS for authentication

## Compliance

**Use for**:

- PCI-DSS: Quarterly vulnerability scanning requirement
- SOC 2: Demonstrate security testing
- HIPAA: Security controls validation
- General best practices

**Limitation**: May not meet all compliance requirements (supplemental scanning needed)

## Exam Focus

### Core Concepts

- Automated vulnerability scanning
- OWASP Top 10 detection
- Pre-production testing
- Continuous monitoring

### Use Cases

- CI/CD integration
- Compliance scanning
- Pre-deployment validation
- Continuous security testing

### Limitations

- No SQLi testing (safety)
- No penetration testing replacement
- Web UI only (not APIs)
- False positives possible
- Limited auth support

### Architecture

- Managed vs custom scans
- Multi-environment strategy
- Integration with Security Command Center
- Defense in depth layer

### Best Practices

- Scan staging first
- Regular scheduled scans
- Combine with manual testing
- Integrate into CI/CD
- Prompt remediation
