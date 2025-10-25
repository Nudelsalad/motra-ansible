# Raspberry Pi Ansible Setup

Streamlined Ansible playbook to provision a Raspberry Pi with:

- **Custom kernel** (from .deb packages)
- **Docker** (via geerlingguy.docker)
- **sched-ext schedulers**
- **motra**

## Prerequisites

- Ansible installed on your control machine
- SSH access to target Raspberry Pi
- Kernel .deb files in `kernel/` directory (if using kernel role)

## Quick Start

### 1. Install Galaxy dependencies

```bash
ansible-galaxy install -r requirements.yml
```

### 2. Configure motra access

Adjust motra github repo URL in `roles/motra/vars/main.yml` if necessary.
If using a private repo, edit your local ssh config (`~/.ssh/config`) to
enable agent forwarding:

```Host your_rpi_hostname_or_ip
    ForwardAgent yes
```

This way no key files need to be copied manually to the Raspberry Pi.

### 2. Configure inventory

Edit `inventory.ini` to point to your Raspberry Pi.

### 3. Run the playbook

```bash
# Full provisioning
ansible-playbook -i inventory.ini site.yml

# Only specific roles
ansible-playbook -i inventory.ini site.yml -t docker,sched_ext
```

## Roles

### kernel

Installs custom kernel from .deb packages (place in `kernel/` directory).

### sched_ext

Installs Rust 1.82+ and sched-ext schedulers from crates.io.

**Default schedulers**:

- scxctl, scxtop, scx_loader, scx_flash, scx_lavd, scx_rusty, scx_bpfland

**Customize** in `vars/prod.yml`:

```yaml
scx_crates_to_install:
  - scxctl
  - scx_flash
```

**Clone scx repo** (optional, for reference):

```bash
ansible-playbook -i inventory.ini site.yml -t sched_ext -e scx_clone_repo=true
```

### motra

Clones and runs motra Docker Compose stack.
