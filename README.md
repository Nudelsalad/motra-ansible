# Raspberry Pi sched-ext harness (minimal)

Small Ansible setup to compare Linux sched-ext schedulers on a two-node Raspberry Pi rig using Docker + cAdvisor + Prometheus + Grafana.

- Server (`rpiserver`): runs the workload (kernel build) and cAdvisor
- Client (`rpiclient`): runs Prometheus/Grafana and owns the file_sd target JSON

## What it does

- Installs Docker and basic tooling on both nodes
- Installs sched-ext v1.0.17 (scx) and `scxctl`
- Starts a constant kernel build on the server, then switches schedulers via `scxctl switch`
- Holds each scheduler for a configurable dwell time, updates Prometheus file_sd with the active scheduler, then reboots the server at the end to stop the build
- Grafana dashboards filter by the plain `scheduler` label

## Prerequisites

- Ansible on your control machine
- SSH access to both Pis; edit `inventory.ini` for IPs and creds

Example inventory:

```
[rpiserver]
raspiserver ansible_host=<server_ip> ansible_user=<user> ansible_password=<pwd> ansible_become_password=<pwd>

[rpiclient]
raspiclient ansible_host=<client_ip> ansible_user=<user> ansible_password=<pwd> ansible_become_password=<pwd>
```

## Quick start

1. Install Galaxy deps

```
ansible-galaxy install -r requirements.yml
```

2. Provision (Docker, scx, motra workspace)

```
ansible-playbook -i inventory.ini site.yml
```

3. Run the scheduler sweep
   Make sure there is a kernel available in the users home folder!

```
ansible-playbook -i inventory.ini plays/sched-matrix.yml
```

4. Motra Reset

```
ansible-playbook -i inventory.ini motra-clean.yml
```

## How it works

- Role `sched-test` on the server starts a background kernel build and iterates schedulers with `scxctl switch -s <sched>`; each step pauses for `sched_test_switch_interval` seconds.
- After each switch it writes `/home/<user>/motra/simple-water-treatment-plant/meta/prometheus/file_sd/target.json` on the client to set a `scheduler` label (used by Grafana).
- When the sweep finishes, the server reboots to terminate the build cleanly.
