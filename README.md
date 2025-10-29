# Raspberry Pi Scheduler Experiment Harness

Ansible automation to provision a two-node Raspberry Pi setup (server + client), build and switch Linux sched-ext schedulers, run a targeted attacker from the client, and collect per-scheduler performance telemetry with Docker + cAdvisor + Prometheus + Grafana.

- Server (rpiserver): runs monitoring (cAdvisor/Redis) and the PLC stack as needed
- Client (rpiclient): runs the attacker container against the server
- Dashboards: provisioned Grafana views with per-scheduler overlays (Thesis Overview, Perf Overview)

## What this does

- Installs Docker and required tooling
- Builds sched-ext from source with a pinned Rust toolchain (nightly-2025-10-28)
- Switches schedulers via `scxctl` with safe fallbacks to direct binaries
- Starts only the monitoring subset of the PLC stack (cAdvisor + Redis) for profiling
- Exposes a “scheduler” label to Prometheus and overlays series by scheduler in Grafana
- Orchestrates experiment runs via a scheduler matrix, optionally with an attacker
- Runs the attacker from the client to generate realistic remote load

## Prerequisites

- Ansible installed on the control machine
- SSH access to both Raspberry Pis (server and client)
- Inventory configured in `inventory.ini`

Optional: if pulling private images or repos, set up SSH agent forwarding locally.

## Inventory

Edit `inventory.ini` to point to your hosts. Example:

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

2. Provision the nodes (Docker, sched-ext, motra workspace)

```
ansible-playbook -i inventory.ini site.yml
```

3. Run the scheduler matrix (server-only orchestration; attacker runs on client)

```
ansible-playbook -i inventory.ini plays/sched-matrix.yml
```

4. Optional cleanup

```
ansible-playbook -i inventory.ini motra-clean.yml
```

## Key playbooks

- `site.yml` — Full provisioning of Docker, sched-ext (from source), and motra workspace
- `plays/sched-matrix.yml` — Runs a matrix of schedulers with/without attacker
  - Attacker is delegated to the client (`rpiclient`) while monitoring runs on the server
  - Forces Docker Compose to recreate monitoring so label changes (scheduler) are reflected
- `motra-clean.yml` — Removes motra working directory and stops related containers

## Roles

- `sched_ext` — Installs rustup (single step, pinned nightly), builds scx from source, provides `scxctl`
- `sched-test` — Writes SCHEDULER into compose .env, switches schedulers (`scxctl` or fallback),
  brings up monitoring (cadvisor, redis), runs the attacker on the client, and performs cleanup

## Observability

- Prometheus scrapes cAdvisor with the container label `scheduler`
- Grafana dashboards use a Scheduler variable and overlay time series by scheduler
- Key panels cover IPC/CPI/MPKI and stall ratios, plus instruction/cycle rates

## Notes

- Attacker runs from the client to generate realistic network load against the server
- Only monitoring services are started by default to keep test noise low
- Compose is forced to recreate monitoring between runs to refresh labels

If a run is interrupted, simply re-run the play; the tasks are idempotent and will converge.
