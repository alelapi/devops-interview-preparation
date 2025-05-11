# Use SBOM - Finding Images Using Cryptolib

## Problem Statement

Given 3 container images, identify which one is using cryptolib by using the BOM (Bill of Materials) binary.

## Solution

### Prerequisites

Ensure you have the `bom` binary installed:

```bash
# Install if needed
go install sigs.k8s.io/bom/cmd/bom@latest

# Or download from releases
curl -L -o bom https://github.com/kubernetes-sigs/bom/releases/download/v0.5.1/bom-amd64-linux
chmod +x bom
sudo mv bom /usr/local/bin/
```

### Step 1: Examine each image directly with BOM and grep for "cryptolib"

Since the task specifically asks to find which image is using "cryptolib," we'll search for this exact term:

```bash
# For this example, let's assume the images are:
# image1: registry.example.com/app1:latest
# image2: registry.example.com/app2:latest
# image3: registry.example.com/app3:latest

# Check image 1
echo "Checking image 1 for cryptolib:"
bom generate -i registry.example.com/app1:latest | grep -i "cryptolib"

# Check image 2
echo "Checking image 2 for cryptolib:"
bom generate -i registry.example.com/app2:latest | grep -i "cryptolib"

# Check image 3
echo "Checking image 3 for cryptolib:"
bom generate -i registry.example.com/app3:latest | grep -i "cryptolib"
```

### Sample Output

The output might look something like this:

```
Checking image 1 for cryptolib:
[No output - cryptolib not found]

Checking image 2 for cryptolib:
[No output - cryptolib not found]

Checking image 3 for cryptolib:
SPDXRef-Package-npm-cryptolib-1.2.3
name: cryptolib
Package: cryptolib
```

### Conclusion

Based on the output, we can determine that **Image 3 (registry.example.com/app3:latest)** is using cryptolib, as it contains specific references to this library.

## Notes

1. In an exam scenario, it's important to search for the exact library mentioned in the question.

2. If no results are found with the exact name, you may need to try variations or check for common abbreviations of the library name:
   ```bash
   bom generate -i image_name | grep -i -E 'cryptolib|crypto-lib|cryptojs|cryptography'
   ```

3. You can also examine the full SBOM output if the direct search doesn't provide results:
   ```bash
   bom generate -i image_name > sbom.txt
   less sbom.txt  # Then search within the file
   ```

4. The `bom` tool can generate output in different formats. If you're having trouble with the default format, try specifying the format explicitly:
   ```bash
   bom generate -f spdx -i image_name | grep -i "cryptolib"
   ```
