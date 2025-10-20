# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Ansible Execution Environment (EE) project for AWX/Ansible Automation Platform, specifically configured for K3s/Kubernetes automation. It uses `ansible-builder` to create a containerized execution environment based on Fedora 42.

## Build and Test Commands

### Building the Execution Environment

```bash
# Primary build method using ansible-builder
ansible-builder build -t awx-ee-k3s:latest .

# Alternative: Build from generated context (after running ansible-builder create)
cd context && docker build -t awx-ee-k3s:latest .
```

### Testing

```bash
# Run test playbook using ansible-navigator
ansible-navigator run test_localhost.yml --execution-environment-image awx-ee-k3s:latest

# Interactive testing
ansible-navigator --execution-environment-image awx-ee-k3s:latest

# Lint playbooks (from venv)
source venv/bin/activate
ansible-navigator lint test_localhost.yml

# View installed collections
ansible-navigator collections --execution-environment-image awx-ee-k3s:latest
```

### Development Workflow

```bash
# 1. Activate virtual environment for local development
source venv/bin/activate

# 2. Make changes to execution-environment.yml

# 3. Rebuild the execution environment
ansible-builder build -t awx-ee-k3s:latest .

# 4. Test changes
ansible-navigator run test_localhost.yml --execution-environment-image awx-ee-k3s:latest
```

## Architecture

### Key Files

- **execution-environment.yml**: Primary configuration file defining the execution environment. This is the main file to edit when modifying the EE.
- **context/**: Auto-generated directory created by `ansible-builder`. Contains Dockerfile and build dependencies. This directory is regenerated during builds.
- **test_localhost.yml**: Simple test playbook that gathers and prints Ansible facts.

### Execution Environment Structure

The `execution-environment.yml` uses version 3 format and defines:

1. **Base Image**: Fedora 42 (`registry.fedoraproject.org/fedora:42`)
2. **Python Dependencies**: ansible-core and ansible-runner (installed via pip)
3. **System Packages**: openssh-clients, sshpass, curl, tar, gzip, git, openssl, ca-certificates
4. **Ansible Collections**: kubernetes.core, community.general
5. **Additional Build Steps**: Helm 3 installation with helm-diff plugin

### Build Process

ansible-builder uses a multi-stage build process:
1. Introspects dependencies from `execution-environment.yml`
2. Generates build context in the `context/` directory
3. Creates Dockerfile with multiple stages
4. Installs system packages, Python packages, and Ansible collections
5. Runs any additional build steps (e.g., Helm installation)

## Deployment

For AWX deployment:

```bash
# Tag and push to registry
docker tag awx-ee-k3s:latest your-registry/awx-ee-k3s:latest
docker push your-registry/awx-ee-k3s:latest
```

Then configure in AWX UI:
- Create Execution Environment with the image URL
- Set pull policy (Always/Missing)
- Assign to job templates

## Customization Guidelines

### Adding Ansible Collections

Edit the `dependencies.galaxy.collections` section in `execution-environment.yml`:

```yaml
galaxy:
  collections:
  - name: kubernetes.core
  - name: community.general
  - name: your.new.collection
```

### Adding System Packages

Add to the `dependencies.system` list in `execution-environment.yml`:

```yaml
system:
- openssh-clients
- sshpass
- your-new-package
```

### Adding Python Packages

Create `context/_build/requirements.txt` (if it doesn't exist) or add via prepend build steps in `execution-environment.yml`.

### Modifying Helm or Additional Tools

Edit the `additional_build_steps.append_final` section in `execution-environment.yml` to add/modify RUN commands for installing additional tools.
