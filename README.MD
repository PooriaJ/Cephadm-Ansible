# Ceph Cluster Setup using Cephadm

This repository contains Ansible playbooks to set up a Ceph cluster using `cephadm`. These playbooks automate the process of deploying a Ceph cluster with necessary roles, variables, and configurations.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Server Requirements](#server-requirements)
- [Setup Instructions](#setup-instructions)
  - [Step 1: Clone Repository](#step-1-clone-repository)
  - [Step 2: Edit Variables](#step-2-edit-variables)
  - [Step 3: Modify `/etc/hosts`](#step-3-modify-etchosts)
  - [Step 4: Configure Inventory](#step-4-configure-inventory)
  - [Step 5: Run the Playbook](#step-5-run-the-playbook)
- [Roles](#roles)
- [Contributing](#contributing)
- [License](#license)

## Overview

This repository is designed to automate the deployment of a Ceph cluster using `cephadm`. The playbooks include all necessary steps from installing Ceph dependencies to configuring the cluster. You can modify inventory and variables files to adapt the configuration to your specific environment.

## Prerequisites

Before using these playbooks, ensure you have the following prerequisites:
- Ansible installed on your control node
- SSH access to all nodes in the Ceph cluster
- Root or sudo access on all nodes
- A **private network** between all servers for internal communication between nodes

> **Note**: `cephadm` will be installed automatically during the playbook run on the cluster nodes.

## Server Requirements

You will need **8 servers** to set up the Ceph cluster:

- **3 MON (Monitor) servers**
- **3 OSD (Object Storage Daemon) servers**
- **2 RGW (Rados Gateway) servers**

Minimum hardware requirements for each server:
- **CPU**: 2 cores
- **RAM**: 2 GB
- **Root Volume**: 25 GB
- **Private Network**: A dedicated network for internal communication between the nodes in the Ceph cluster

In addition to the root volume, **each OSD server** must have **an additional disk** attached for use as OSD storage devices. These disks should be properly defined in the playbook variables.

## Setup Instructions

### Step 1: Clone Repository

Clone this repository onto your control node:

```bash
git clone https://github.com/PooriaJ/Cephadm-Ansible.git
cd Cephadm-Ansible
```

### Step 2: Edit Variables

Edit the necessary variables in the `vars` directory before running the playbook:

- `osd_disk_path`: Path to the OSD disks (e.g., `/dev/vdb`, `/dev/vdc`, `/dev/vdd`)
- `rgw_port`: The port to use for RGW (e.g., `80`)
- `name_of_rgw`: The name of the RGW service (e.g., `mainrgw`)
- `first_client`: Name of the first Ceph client (e.g., `pooria`)
- `ceph_exporter_version`: Version of the Ceph exporter (e.g., `latest`)
- `node_exporter_version`: Version of the node exporter (e.g., `latest`)
- `name_server`: The name server IP (e.g., `8.8.8.8`)
- `cluster_network`: The **private network** range for internal communication (e.g., `192.168.1.0/24`)

These variables should be defined in the `vars/` files.

### Step 3: Modify `/etc/hosts`

Before running the playbook, you need to modify the `etchost.yaml` file in the `vars/` directory to match the IP addresses of your Ceph cluster nodes.

The file is located at `vars/etchost.yaml` and contains the IP addresses and hostnames for the Ceph cluster. Update the IP addresses to match the private IPs of your nodes.

Example of the `etchost.yaml` file structure:

```yaml
127.0.0.1 localhost
192.168.1.42 mon-0
192.168.1.22 mon-1
192.168.1.158 mon-2

192.168.1.105 osd-1
192.168.1.137 osd-0
192.168.1.227 osd-2

192.168.1.57 rgw-0
192.168.1.183 rgw-1

ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Update the above file to reflect the private IP addresses of your MON, OSD, and RGW nodes, replacing the sample IPs with your own.

### Step 4: Configure Inventory

Update the `inventory.yaml` file to define the nodes that will be part of your Ceph cluster. For example:

```yaml
all:
  children:
    cluster:
      hosts:
        mon-0:
          ansible_host: 1.1.1.1
          init_cluster: 'true'
        mon-1:
          ansible_host: 1.1.1.2
        mon-2:
          ansible_host: 1.1.1.3
```

Ensure that the inventory includes separate hosts for MON, OSD, and RGW services, distributed across the servers, and that each server has access to the private network.

### Step 5: Run the Playbook

Run the `setup.yaml` playbook to install and configure the Ceph cluster:

```bash
ansible-playbook -i inventory.yaml setup.yaml
```

This will install Ceph, configure your cluster, set up the private network communication, and start the necessary services on the specified nodes.

## Roles

The playbooks use Ansible roles to divide tasks into reusable components. You can find the roles inside the `roles/` directory, where each role corresponds to specific steps in the Ceph deployment process.

## Contributing

If you'd like to contribute, feel free to fork the repository and submit a pull request with your changes.

## License

This project is licensed under the MIT License. See the [LICENSE](./LICENSE) file for details.