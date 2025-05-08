# Image Scanning and Security

## Introduction

As software supply chain attacks increase in frequency and sophistication, organizations are seeking effective ways to secure their software development lifecycle. Two critical components in this effort are Software Bills of Materials (SBOMs) and vulnerability scanning. This guide focuses on Trivy, a powerful open-source scanner, and how it can be used to generate and analyze SBOMs for enhanced security posture.

## Understanding SBOM

### What is an SBOM?

A Software Bill of Materials (SBOM) is a formal, machine-readable inventory of software components and dependencies, information about those components, and their hierarchical relationships. Think of it as a comprehensive ingredient list for software, similar to the nutrition label on food products.

An SBOM provides transparency into:
- **Components**: All libraries, modules, and packages included in the software
- **Versions**: Specific versions of each component
- **Dependencies**: Relationships between components
- **Suppliers**: Origin of components
- **Licensing**: License information for each component

### SBOM Formats

There are several SBOM formats available, with the following being the most widely adopted:

#### 1. CycloneDX

CycloneDX is a lightweight SBOM specification designed by OWASP for application security contexts and supply chain component analysis.

**Key characteristics:**
- Designed specifically for cybersecurity use cases
- Supports multiple formats: JSON, XML, Protocol Buffers
- Lightweight and focused on security-relevant metadata
- Includes vulnerability reporting capabilities
- Supports VEX (Vulnerability Exploitability eXchange)

**Sample CycloneDX JSON (simplified):**

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.4",
  "serialNumber": "urn:uuid:3e671687-395b-41f5-a30f-a58921a69b79",
  "version": 1,
  "components": [
    {
      "type": "library",
      "name": "acme-library",
      "version": "1.0.0",
      "purl": "pkg:npm/acme-library@1.0.0"
    }
  ]
}
```

#### 2. SPDX (Software Package Data Exchange)

SPDX is an open standard for communicating software bill of materials information, including components, licenses, copyrights, and security references.

**Key characteristics:**
- ISO/IEC 5962:2021 standard
- Comprehensive license information
- Detailed metadata capabilities
- Mature standard with strong industry adoption
- Originally focused on license compliance, now expanded for security

**Sample SPDX JSON (simplified):**

```json
{
  "spdxVersion": "SPDX-2.2",
  "dataLicense": "CC0-1.0",
  "SPDXID": "SPDXRef-DOCUMENT",
  "name": "example-project",
  "documentNamespace": "http://spdx.org/spdxdocs/example-project",
  "packages": [
    {
      "name": "acme-library",
      "SPDXID": "SPDXRef-Package-1",
      "versionInfo": "1.0.0",
      "downloadLocation": "https://example.com/acme-library-1.0.0.tar.gz",
      "licenseConcluded": "MIT"
    }
  ]
}
```

#### 3. SWID (Software Identification Tags)

SWID tags provide identification information for software, primarily used for inventory and asset management.

**Key characteristics:**
- Standardized by ISO/IEC 19770-2
- Focused on software identification and inventory
- Less detailed than CycloneDX or SPDX
- Primarily used for IT asset management

## Introduction to Trivy

### What is Trivy?

Trivy is a comprehensive and versatile security scanner developed by Aqua Security. It is designed to find vulnerabilities, misconfigurations, secrets, and generate SBOMs across various targets including containers, filesystems, git repositories, and Kubernetes clusters.

## Working with Trivy

### Basic Commands

Trivy has several subcommands for different scanning targets:

- `image`: Scan container images
- `filesystem` (or `fs`): Scan local filesystem
- `repository` (or `repo`): Scan a remote git repository
- `kubernetes` (or `k8s`): Scan Kubernetes resources
- `config`: Scan IaC configuration files
- `sbom`: Scan SBOM files

### Scanning Container Images

#### Basic Image Scanning

```bash
# Scan an image for vulnerabilities (pull from remote registry)
trivy image nginx:latest

# Scan a locally built image
trivy image my-local-image:tag

# Scan and filter by severity
trivy image --severity HIGH,CRITICAL nginx:latest

# Output results in JSON format
trivy image --format json --output results.json nginx:latest
```

#### Advanced Image Scanning Options

```bash
# Ignore unfixed vulnerabilities
trivy image --ignore-unfixed nginx:latest

# Include package information in the report
trivy image --list-all-pkgs nginx:latest

# Scan with custom policy
trivy image --policy=./policy/ nginx:latest

# Show dependency tree for vulnerabilities
trivy image --dependency-tree nginx:latest
```

### Scanning Kubernetes Clusters

```bash
# Scan a Kubernetes cluster (requires kubectl context)
trivy k8s --report summary cluster

# Scan a specific namespace
trivy k8s --namespace default all

# Scan a specific workload
trivy k8s deployment/my-deployment
```

### Scanning Infrastructure as Code

```bash
# Scan IaC files for misconfigurations
trivy config ./terraform/

# Scan Kubernetes YAML files
trivy config ./kubernetes-manifests/

# Scan with custom policies
trivy config --policy=./policy/ ./terraform/
```

## Generating SBOMs with Trivy

### SBOM Output Formats

Trivy can generate SBOMs in the following formats:

- **CycloneDX**: Using `--format cyclonedx`
- **SPDX**: Using `--format spdx-json` or `--format spdx`

### Command Examples

#### Generate CycloneDX SBOM for a Container Image

```bash
# Generate a CycloneDX SBOM for a container image
trivy image --format cyclonedx --output sbom.cdx.json nginx:latest

# Include full dependency tree
trivy image --format cyclonedx --output sbom.cdx.json --dependency-tree nginx:latest

# Include package info for all packages
trivy image --format cyclonedx --output sbom.cdx.json --list-all-pkgs nginx:latest
```

