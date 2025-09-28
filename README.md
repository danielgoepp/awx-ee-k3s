# AWX Execution Environment - K3s

Custom Ansible Execution Environment for AWX/Ansible Automation Platform with Kubernetes support.

## Overview

This project builds a containerized Ansible execution environment that includes:
- Ansible Core and Ansible Runner
- Kubernetes.core collection for Kubernetes automation
- SSH connectivity tools (openssh-clients, sshpass)
- Based on the official AWX execution environment

## Quick Start

### Prerequisites
- Docker or Podman
- ansible-builder
- ansible-navigator (optional, for testing)

### Build the Execution Environment

```bash
# Build using ansible-builder
ansible-builder build -t awx-ee-k3s:latest .

# Or build directly with Docker
cd context && docker build -t awx-ee-k3s:latest .
```

### Test the Environment

```bash
# Run the test playbook
ansible-navigator run test_localhost.yml --execution-environment-image awx-ee-k3s:latest

# Or run interactively
ansible-navigator --execution-environment-image awx-ee-k3s:latest
```

## Configuration

### Execution Environment Definition
The `execution-environment.yml` file defines:
- **Base Image**: `quay.io/ansible/awx-ee:latest`
- **Python Packages**: ansible-core, ansible-runner
- **System Packages**: openssh-clients, sshpass
- **Collections**: kubernetes.core

### Customizing Dependencies

#### Adding Ansible Collections
Edit `execution-environment.yml`:
```yaml
dependencies:
  galaxy:
    collections:
    - name: kubernetes.core
    - name: your.new.collection  # Add new collections here
```

#### Adding System Packages
Edit `context/_build/bindep.txt`:
```
openssh-clients
sshpass
your-new-package  # Add new system packages here
```

#### Adding Python Packages
Create `context/_build/requirements.txt`:
```
some-python-package==1.0.0
```

## Usage in AWX

1. Build and push the image to your container registry:
   ```bash
   ansible-builder build -t your-registry/awx-ee-k3s:latest .
   docker push your-registry/awx-ee-k3s:latest
   ```

2. In AWX, create a new Execution Environment:
   - Name: `AWX EE K3s`
   - Image: `your-registry/awx-ee-k3s:latest`
   - Pull: `Always` or `Missing`

3. Assign the execution environment to your job templates

## Development

### Local Testing
```bash
# Activate virtual environment
source venv/bin/activate

# Lint playbooks
ansible-navigator lint test_localhost.yml

# Test collections
ansible-navigator collections --execution-environment-image awx-ee-k3s:latest
```

### Rebuilding After Changes
```bash
# Rebuild the execution environment
ansible-builder build -t awx-ee-k3s:latest .

# Test changes
ansible-navigator run test_localhost.yml --execution-environment-image awx-ee-k3s:latest
```

## Project Structure

```
.
├── execution-environment.yml    # EE definition file
├── context/                     # Generated build context
│   ├── Dockerfile              # Generated multi-stage Dockerfile
│   └── _build/                 # Build dependencies
│       ├── requirements.yml    # Ansible collections
│       ├── bindep.txt         # System packages
│       └── scripts/           # Build scripts
├── test_localhost.yml          # Test playbook
└── venv/                      # Python virtual environment
```

## Troubleshooting

### Build Issues
- Ensure Docker/Podman is running
- Check `ansible-builder --version` compatibility
- Verify base image accessibility: `docker pull quay.io/ansible/awx-ee:latest`

### Runtime Issues
- Check execution environment logs in AWX
- Verify required collections are installed: `ansible-navigator collections`
- Test playbook syntax: `ansible-navigator lint your-playbook.yml`

### Permission Issues
- The container runs as user ID 1000
- Ensure proper volume mounts for file access
- Check SELinux contexts if applicable

## License

This project follows the same license as the base AWX execution environment.