# Rocky Linux Hardening Automation (Ansible)

I'm a student learning cybersecurity, and I built this as part of a home lab project. It's an Ansible playbook that hardens a Rocky Linux 10 server automatically.

I first did all the hardening by hand. Then I turned those steps into this playbook so they can be repeated on any server without doing it manually each time. That's the main thing I wanted to learn: how to automate security setup instead of clicking through it every time.

## What it does

| Area | What it does |
| --- | --- |
| Updates | Installs all available patches |
| Cleanup | Removes unused software (Samba) |
| SSH | Key-only login, moves SSH to port 2222, blocks root login, limits login tries |
| Firewall | Opens only port 2222, removes services I don't need, blocks ICMP timestamps |
| fail2ban | Bans IPs that fail SSH login too many times |
| Auditing | Turns on auditd and adds rules to track important file changes |
| Hygiene | Adds login warning banners, locks down cron permissions, disables an unused module (TIPC) |

## How it works

Ansible runs from one machine (my Kali box) and configures the other machine (Rocky) over SSH. Rocky doesn't need anything installed on it except SSH and Python, which it already has.

## How to run it

1. Set the target server in inventory.ini.
2. Do a test run first (shows what would change without changing anything): ansible-playbook hardening.yml --check -K
3. Run it for real: ansible-playbook hardening.yml -K

The -K asks for the sudo password.

## A note on re-running

It's safe to run this more than once. If the server is already set up, the playbook just reports "ok" and skips it instead of redoing the work. I learned this is called being "idempotent."

## About this project

This playbook is one part of a bigger home lab I built. The full project also includes vulnerability scanning with Nessus, a Wazuh SIEM for detecting attacks, and CIS benchmark hardening. This piece shows how I took the manual hardening and automated it.

## Note

I built this for an isolated lab. If you run it somewhere else, read through the tasks first. Some of them (like changing the SSH port) could lock you out if the settings don't match your setup.

## Results

Checked the work with before-and-after scans

**Vulnerability scan (Nessus, credentialed)**

| Severity      | Before | After |
| ------------- | ------ | ----- |
| Critical      | 2      | 0     |
| High          | 5      | 0     |
| Low           | 1      | 0     |
| Actionable    | 8      | 0     |

The baseline had 8 actionable findings (2 Critical, 5 High, 1 Low) plus 49 informational, 57 total. Every Critical and High was a missing OS security patch that only showed up once Nessus authenticated over SSH. After hardening, the same credentialed scan came back with 0 across every severity. The only results left are informational (system enumeration data), not vulnerabilities.

**CIS Benchmark (Wazuh SCA, CIS Rocky Linux 10 v1.0.0)**

Score went from 57% (91 controls passed) to 66% (105 passed). The remaining failures are partition-layout controls that need an install-time disk scheme, documented as a known, scoped gap.

**Automation (Ansible dry run)**

The `--check` run against the hardened host completed with 0 failures (ok=23, changed=7), confirming the playbook reproduces the full configuration and is idempotent.

Full write-up with screenshots: https://easy-delphinium-785.notion.site/Linux-Hardening-Vulnerability-Assessment-Detection-Lab-3816f3a66a5381eeaf99e291eb62842d?pvs=73
