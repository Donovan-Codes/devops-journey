# Phase 1 — Linux Fundamentals

Environment: Ubuntu Server LTS in a local VM, no GUI.

## Labs

| # | Lab | Status |
|---|---|---|
| 1 | Key-based SSH + disable password auth | ⚪ |
| 2 | Users, groups, shared directory permissions | ⚪ |
| 3 | Write a systemd service | ⚪ |
| 4 | Serve nginx on a custom port through a firewall | ⚪ |
| 5 | Log analysis with grep/awk/sort/uniq | ⚪ |
| 6 | Cron job with logging | ⚪ |
| 7 | Timed bare-install-to-serving rebuild | ⚪ |

## Break-and-fix drills

| # | Drill | Status |
|---|---|---|
| 1 | Fill the disk, find it, recover | ⚪ |
| 2 | Corrupt an nginx config, diagnose from the log alone | ⚪ |
| 3 | Wrong `~/.ssh` permissions, understand why sshd refuses | ⚪ |
| 4 | Kill a managed process, observe systemd | ⚪ |
| 5 | Break DNS, distinguish network failure from name resolution failure | ⚪ |
| 6 | Bad `/etc/fstab` entry, recover from the emergency shell | ⚪ |

For each drill, record in the weekly log: **the symptom**, the command that revealed
the cause, and the fix. The symptom is the part that matters — that's what you'll
actually be handed in a real incident.

## VM details

| | |
|---|---|
| Hypervisor | |
| Ubuntu version | |
| Resources | vCPU / RAM / disk |
| Networking mode | NAT + port forward / bridged |
| SSH command | |
