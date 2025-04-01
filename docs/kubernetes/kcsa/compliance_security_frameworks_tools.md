# Compliance and Security frameworks

## Distribute - Container manifest

### Kubesec
A security risk analysis tool for Kubernetes resources that scans YAML manifests and Helm charts to identify security misconfigurations, vulnerabilities, and non-compliance with best practices.

### Terrascan
An open-source security scanner that detects compliance and security violations across Infrastructure as Code (IaC) tools like Terraform, Kubernetes, Helm, and Dockerfiles to help enforce security best practices.

## Distribute - Security Tests

### Nuclei
A fast, template-based vulnerability scanner that detects security issues across various attack surfaces by using customizable templates for targeted scanning.

### Trivy
An all-in-one open-source scanner for containers, filesystems, Git repositories, and Kubernetes that detects vulnerabilities, misconfigurations, and secrets.

### Snyk
A developer security platform that finds, fixes, and monitors vulnerabilities in application code, open source dependencies, containers, and infrastructure as code.

### Clair
An open-source container vulnerability scanner that statically analyzes container images for known security vulnerabilities in application dependencies.

### Grype
A vulnerability scanner for container images and filesystems created by Anchore that provides fast, comprehensive detection of package vulnerabilities.

### Sysdig Secure
A cloud-native security platform designed for securing containerized applications, Kubernetes environments, and cloud workloads. It provides comprehensive security capabilities across the entire container lifecycle.

## Distribute - Signing and Trust

### in-toto
A framework that cryptographically ensures software supply chain integrity by tracking each step in the development process with signed metadata to verify the chain hasn't been compromised.

### Notation
A CNCF project for signing and verifying OCI container images and artifacts with a pluggable signature verification architecture.

### TUF (The Update Framework): 
A secure framework for software updates that protects against various attacks by using multiple layers of signing keys and metadata verification.

### Sigstore
An open-source project providing free tools for code signing, transparency, and verification to secure software supply chains without managing private keys.

## Deploy - Pre-flight checks

### Gatekeeper
A Kubernetes admission controller using Open Policy Agent (OPA) that enforces configurable policies to validate, mutate, or reject resources before they're admitted to the cluster.

### Kyverno
A policy engine designed specifically for Kubernetes that can validate, mutate, and generate resources using policy as code without requiring a new language, providing native YAML/JSON support.

## Distribution - Response & Investigation

### Wazuh
An open-source security monitoring platform that provides threat detection, integrity monitoring, and compliance capabilities through log analysis, file integrity checking, and security alerts.

### Snort
A widely-used open-source network intrusion detection and prevention system (IDPS) that performs real-time traffic analysis and packet logging to detect and prevent network attacks.

### Zeek
A powerful network security monitoring tool that analyzes network traffic to identify suspicious activity by generating high-level logs about network behavior rather than just matching signatures.

## Runtime - Orchestration

### kube-bench
An open-source tool that checks whether Kubernetes deployments align with CIS Kubernetes Benchmark security recommendations by automatically testing control plane and worker nodes for best practices.

### Trivy
An all-in-one vulnerability scanner for containers, filesystems, Git repositories, and Kubernetes that detects vulnerabilities, misconfigurations, and secrets.

### Falco
A cloud-native runtime security tool that detects abnormal behavior and security threats in real-time by monitoring Linux system calls, container activities, and Kubernetes events.

### SPIFFE 
A set of open-source standards for securely identifying and authenticating services to each other across heterogeneous environments using platform-agnostic, cryptographic identities.

## Runtime - Storage

### Rook
A cloud-native storage orchestrator for Kubernetes that automates deployment, bootstrapping, configuration, scaling, and recovery of storage services like Ceph.

### Ceph
A highly scalable, distributed storage system providing object, block, and file storage in a unified platform designed for high performance, reliability, and no single point of failure.

### Gluster
An open-source distributed file system that can scale to several petabytes, handles thousands of clients, and is suitable for data-intensive workloads like cloud storage and media streaming.

## Runtime - Access

### Keycloak
An open-source identity and access management solution that provides single sign-on, identity federation, social login, and user management with support for standard protocols like OAuth 2.0 and SAML.

### Teleport
A security gateway for accessing Kubernetes clusters, servers, and applications that unifies access controls with certificate-based authentication, session recording, and audit logging.

### Vault
HashiCorp's secrets management tool that securely stores and tightly controls access to tokens, passwords, certificates, and encryption keys with dynamic secrets generation, data encryption, and leasing/renewal capabilities.
