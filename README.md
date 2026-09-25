# Home SOC Lab

A self-built Security Operations Center lab for hands-on detection engineering practice, running entirely on a MacBook Air (Apple Silicon) using VMware Fusion.

## Overview

This lab simulates a small monitored network: an isolated attacker machine, a monitored target, and a SIEM collecting and alerting on activity between them. Built from scratch as part of my transition into SOC analyst work, alongside TryHackMe's SAL1 pathway.

## Architecture

                VMware Fusion (Apple Silicon / ARM64)
                          |
          Private lab network: 10.10.10.0/24
                          |
    +---------------------+---------------------+
    |                                            |
Kali Linux (attacker) Ubuntu Server (target/victim)
10.10.10.15 10.10.10.10
| |
| Wazuh Agent installed
| reports to manager
| |
+----------------------> Wazuh Manager <-----+
10.10.10.20
(Ubuntu Server 26.04 LTS,
native install manager,
indexer, and dashboard)

Each VM sits on an isolated private network, with a separate NAT interface for internet access. No lab traffic touches the home network.

## What's running

- Wazuh Manager, Indexer, Dashboard - installed natively on Ubuntu Server 26.04 LTS (ARM64)
- Wazuh Agent - deployed on the Ubuntu target, reporting host events back to the manager
- Kali Linux - attacker machine, set up for generating test traffic

## A real problem I hit and fixed: ARM64 vs. Docker

My first attempt ran Wazuh via Docker on the Ubuntu Server VM. It failed with a Signal 11 segmentation fault in the manager container on startup, which cascaded into the dashboard throwing ECONNREFUSED.

Root cause: Wazuh's official Docker images are built for x86_64. Running them on Apple Silicon (ARM64) forces QEMU binary translation inside the container, and Wazuh's manager does low-level operations that don't survive that translation layer cleanly.

Fix: Dropped Docker entirely and installed Wazuh natively on the ARM64 Ubuntu VM using the official install script. Since the host OS itself is genuinely ARM64, the native .deb packages install and run cleanly.

Along the way I also worked through:
- A disk space exhaustion mid-install
- A corrupted dpkg package state from repeated interrupted install attempts

## Skills demonstrated

- Linux system administration (Ubuntu Server, systemd, disk/LVM management)
- Network segmentation and static IP configuration across isolated VLANs
- SIEM deployment and troubleshooting
- Root-cause diagnosis across architecture, containerization, and package management layers

## Next steps

- [ ] Simulate an SSH brute-force attempt and capture the resulting Wazuh detection
- [ ] Add Windows endpoint with Sysmon
- [ ] Write custom Wazuh detection rules
- [ ] Add Suricata for network-level IDS
- [ ] Script alert triage/enrichment via the Wazuh API (Python)
