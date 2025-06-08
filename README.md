# Project: Distributed PostgreSQL Database with Streaming Replication

**Course:** INT14148 - Distributed Database Systems  
**Project ID:** Project 12  
**Student:** Huynh Minh Thinh  
**Student ID:** N22DCCN082  
**Class ID:** D22CQCN01-N

---

## 📖 Table of Contents

- [Project Description](#-project-description)
- [System Architecture](#-system-architecture)
- [Technologies Used](#-technologies-used)
- [Setup and Configuration](#-setup-and-configuration)
  - [Prerequisites](#prerequisites)
  - [Step 1: Configure the Primary Server (Lenovo)](#step-1-configure-the-primary-server-lenovo)
  - [Step 2: Configure a Standby Server (Repeat for Each VM)](#step-2-configure-a-standby-server-repeat-for-each-vm)
- [Verification](#-verification)
- [Project Report](#-project-report)

## 📝 Project Description

This project involves the implementation and analysis of a four-node distributed database system using PostgreSQL 17. The primary goal is to build a high-availability cluster using native streaming replication, a standard industry practice for ensuring data redundancy and scaling read operations.

The system is built on a hybrid physical-virtual model, with the host machine acting as the primary server and three KVM virtual machines serving as hot-standby replicas. The cluster is populated with the Pagila sample dataset to provide a realistic schema for analysis. The final report analyzes how this distributed architecture can effectively support a high-traffic web application, such as an online movie rental service.

## 🏗️ System Architecture

The cluster consists of one primary node that handles all write operations (`INSERT`, `UPDATE`, `DELETE`) and three standby nodes that replicate data from the primary in near real-time. The standby nodes are configured as "hot standbys," allowing them to serve read-only queries.

```mermaid
graph TD
    subgraph "Application Layer"
        A(Web Application)
    end

    subgraph "Database Cluster (192.168.122.0/24)"
        P[Primary Server<br>Lenovo<br>192.168.122.1]
        S1[Standby 1<br>node1<br>192.168.122.101]
        S2[Standby 2<br>node2<br>192.168.122.102]
        S3[Standby 3<br>node3<br>192.168.122.103]
    end

    A -- "Writes (SQL)" --> P
    A -- "Reads (SQL)" --> S1
    A -- "Reads (SQL)" --> S2
    A -- "Reads (SQL)" --> S3

    P -- "Streaming Replication (WAL)" ==> S1
    P -- "Streaming Replication (WAL)" ==> S2
    P -- "Streaming Replication (WAL)" ==> S3

    style P fill:#c9f,stroke:#333,stroke-width:2px
    style S1 fill:#9cf,stroke:#333,stroke-width:2px
    style S2 fill:#9cf,stroke:#333,stroke-width:2px
    style S3 fill:#9cf,stroke:#333,stroke-width:2px
```

| Role      | Hostname | Operating System    | Static IP Address |
| :-------- | :------- | :------------------ | :---------------- |
| Primary   | Lenovo   | EndeavourOS (Host)  | `192.168.122.1`   |
| Standby 1 | node1    | Ubuntu Server 24.04 | `192.168.122.101` |
| Standby 2 | node2    | Ubuntu Server 24.04 | `192.168.122.102` |
| Standby 3 | node3    | Ubuntu Server 24.04 | `192.168.122.103` |

## 🛠️ Technologies Used

- **DBMS:** PostgreSQL 17
- **Replication:** PostgreSQL Native Streaming Replication
- **Virtualization:** KVM/QEMU
- **Management:** `virt-manager`, `libvirt`, `virsh`
- **Host OS:** EndeavourOS (Arch-based Linux)
- **VM OS:** Ubuntu Server 24.04 LTS
- **Dataset:** Pagila
- **Report Tool:** LaTeX

## ⚙️ Setup and Configuration

This guide details the steps to build the replicated cluster from scratch.

### Prerequisites

1.  A host machine with KVM/QEMU and `virt-manager` installed.
2.  Three Ubuntu Server 24.04 VMs created and configured with static IPs and hostnames as per the table above.
3.  PostgreSQL 17 installed on the host machine and on all three VMs.
4.  The Pagila sample database loaded into the primary server's PostgreSQL instance.

### Step 1: Configure the Primary Server (`Lenovo`)

Prepare the primary server to accept replication connections.

1.  **Create a Replication User:**

    ```sql
    CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'your_secure_password';
    ```

2.  **Edit `postgresql.conf`:**
    Set the following parameters to enable replication broadcasting.

    ```ini
    # /var/lib/postgres/data/postgresql.conf
    listen_addresses = 'localhost, 192.168.122.1'
    wal_level = replica
    max_wal_senders = 10
    ```

3.  **Edit `pg_hba.conf`:**
    Add this rule to the bottom of the file to allow the `replicator` user to connect from the standby network.

    ```ini
    # /var/lib/postgres/data/pg_hba.conf
    # TYPE   DATABASE      USER        ADDRESS             METHOD
    host     replication   replicator  192.168.122.0/24    scram-sha-256
    ```

4.  **Restart the Primary Server:**
    Apply the changes by restarting the PostgreSQL service.

    ```bash
    sudo systemctl restart postgresql.service
    ```

### Step 2: Configure a Standby Server (Repeat for Each VM)

The following steps must be performed on `node1`, `node2`, and `node3`.

1.  **Clean Up Environment:**

    ```bash
    # Stop any running postgres service
    sudo systemctl stop postgresql

    sudo rm -rf /var/lib/postgresql/17/main/
    ```

2.  **Generate Required Locale:**
    The primary database was initialized with `en_US.UTF-8`. Ensure this locale exists on the standby OS.

    ```bash
    # Uncomment en_US.UTF-8 in /etc/locale.gen
    sudo sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen

    # Generate the locale
    sudo locale-gen
    ```

3.  **Perform Base Backup:**
    Use `pg_basebackup` to clone the primary's data directory. Run this command as the `postgres` system user.

    ```bash
    sudo -u postgres pg_basebackup -h Lenovo -U replicator \
    -D /var/lib/postgresql/17/main -P -R -Xs
    ```

    > **Note:** The `-R` flag is crucial as it automatically creates the `standby.signal` file and writes the `primary_conninfo` connection settings into `postgresql.auto.conf`.


4.  **Enable and Start the Service:**
    Use the version-specific service name to enable and start PostgreSQL.

    ```bash
    sudo systemctl enable postgresql
    sudo systemctl start postgresql
    ```

## ✅ Verification

After configuring all three standbys, connect to `psql` on the **primary server (`Lenovo`)** and run the following query:

```sql
SELECT client_addr, state, sent_lsn, replay_lsn
FROM pg_stat_replication;
```

You should see **three rows**, one for each standby (`192.168.122.101`, `102`, `103`), with the `state` listed as `streaming`. This confirms the cluster is fully operational.

## 📄 Project Report

The final, detailed academic report for this project can be found in this repository:

[**View the Full PDF Report**](./LiteratureReviewTemplate_INT1414.pdf)

---

**Author:** Huynh Minh Thinh  
**Date:** June 2025
