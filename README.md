# Project: Distributed PostgreSQL Database with Streaming Replication

**Course:** INT14148 - Distributed Database Systems  
**Project ID:** Project 12  
**Student ID** N22DCCN082  
**Class ID:** D22CQCN01-N

## Project Description

This project involves the implementation and analysis of a four-node distributed database system using PostgreSQL. The primary goal is to build a high-availability cluster using native streaming replication, a standard industry practice for ensuring data redundancy and scaling read operations.

The system is built on a hybrid physical-virtual model, with the host machine acting as the primary server and three KVM virtual machines serving as hot-standby replicas. The cluster is populated with the Pagila sample dataset to provide a realistic schema for analysis. The final report analyzes how this distributed architecture can effectively support a high-traffic web application, such as an online movie rental service.

## System Architecture

The cluster consists of one primary node that handles all write operations and three standby nodes that replicate data from the primary in near real-time. The standby nodes can be used to serve read-only queries.

| Role      | Hostname | Operating System    | Static IP Address |
| :-------- | :------- | :------------------ | :---------------- |
| Primary   | Lenovo   | EndeavourOS (Host)  | `192.168.122.1`   |
| Standby 1 | node1    | Ubuntu Server 24.04 | `192.168.122.101` |
| Standby 2 | node2    | Ubuntu Server 24.04 | `192.168.122.102` |
| Standby 3 | node3    | Ubuntu Server 24.04 | `192.168.122.103` |

## Technologies Used

- **DBMS:** PostgreSQL 17
- **Replication:** PostgreSQL Native Streaming Replication
- **Virtualization:** KVM/QEMU
- **Management:** `virt-manager`, `libvirt`, `virsh`
- **Host OS:** EndeavourOS (Arch-based Linux)
- **VM OS:** Ubuntu Server 24.04 LTS
- **Dataset:** Pagila
- **Report Tool:** LaTeX

## Setup and Deployment Overview

The environment was configured following these major phases:

#### 1. Primary Node Setup (Host Machine)

- PostgreSQL 17 was installed on the EndeavourOS host.
- The configuration files (`postgresql.conf` and `pg_hba.conf`) were modified to enable replication by setting `wal_level = replica`, updating `listen_addresses`, and adding host-based authentication rules for a dedicated replication user.
- The Pagila dataset was imported into a new database named `pagila`.

#### 2. Standby Node Template & Cloning

- A "golden image" template was created using a minimal installation of Ubuntu Server 24.04 on a KVM virtual machine.
- The template was fully updated (`apt update && apt upgrade`), and essential tools, including the matching PostgreSQL 17 version, were pre-installed.
- This template was then cloned three times to create the `node1`, `node2`, and `node3` virtual machines.

#### 3. Network & Identity Configuration

- A private virtual network (`192.168.122.0/24`) was used for all nodes.
- Each cloned VM was configured with a unique hostname and a unique static IP address to prevent network conflicts.
- Hostname resolution was configured on all four nodes via the `/etc/hosts` file to allow for easy communication by name.
