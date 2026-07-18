# Edge Fleet Update Automation

This repository contains an Ansible playbook and inventory configuration designed to safely manage system updates across a fleet of headless Debian-based edge machines.

## Features

* **Rolling Updates (`serial: 30%`):** Updates machines in dynamically sized batches to prevent total network downtime. If a batch fails, the playbook halts to protect the rest of the fleet.
* **Offline Node Tolerance (`ignore_unreachable`):** Gracefully skips offline or sleeping edge devices without throwing fatal errors or halting the playbook.
* **Storage Optimization:** Automatically removes outdated kernels and unneeded dependencies (`autoremove`) to preserve limited disk space on edge devices.
* **Conditional Reboots:** Only initiates a reboot if a core system update requires it (verifies against `/var/run/reboot-required`).

## Prerequisites

1. Modern Ansible Installation
To avoid Python compatibility errors (`six.moves`) on newer operating systems like Debian 12/13, this playbook requires a modern version of Ansible (Core 2.15+) installed via `pipx`.

```bash
# Remove outdated apt versions
sudo apt remove ansible -y
sudo apt autoremove -y
```
```bash
# Ensure pip is up to date
python3 -m pip install --user --upgrade pip
```
```bash
# Install modern Ansible
sudo apt update
python3 -m pip install --user ansible
```
```bash
# If installing on Raspberry Pi or you receive a hash mismatch error
python3 -m pip install --user --no-cache-dir --index-url https://pypi.org/simple ansible
```
2. SSH Key Authentication

The control machine must have its public SSH key distributed to all target edge nodes. Passwords are not used for initial connection.
```bash
# To copy your key to a new node:
ssh-copy-id -i ~/.ssh/id_rsa username@<NODE_IP>
```

To prevent Ansible from hanging on new SSH fingerprint prompts, create an ansible.cfg file in the same directory as your playbook:

[defaults]
host_key_checking = False

## Project Structure

    hosts.yaml - The inventory file defining the edge machines, their IP addresses, and group variables.

    update.yml - The main playbook containing the update, upgrade, and reboot tasks.

  
## Usage
1. Verify Connectivity

Before running an update, test the connection to your edge fleet.
The playbook uses gather_facts: no, so running a manual ping test ensures all active machines are reachable.
```bash
ansible -i hosts.yaml edge_machines -m ping
```
2. Run the Update Playbook

Execute the playbook against the inventory. If the user requires a password to execute sudo commands on the edge machines, append the -K flag.
```bash
ansible-playbook -i hosts.yaml update.yml -K
```
3. Targeting Specific Nodes

To test the playbook on a single machine (e.g., edge_machine_1) before rolling it out to the entire group:
```bash
ansible-playbook -i hosts.yaml update.yml --limit edge_machine_1 -K
```
