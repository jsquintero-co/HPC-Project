# High-Performance Computing (HPC) Cluster Documentation

## 1. Overview
This document outlines the architecture, hardware setup, network configuration, parallel processing stack, remote access setup, and monitoring infrastructure for a two-node heterogeneous High-Performance Computing (HPC) cluster.

The cluster operates on a distributed computing framework utilizing OpenMPI over a secure, private network layer, allowing synchronized workload execution across both nodes.

---

## 2. Hardware Specifications

| Node Role | Identifier | CPU | Cores / Threads | Clock Speed | RAM | Storage | GPU | Operating System |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Master Node (Tower)** | `hpc1` | Intel Core i5-10400F | 6C / 12T | Up to 4.30 GHz | 24 GB DDR4 | 700 GB | NVIDIA GTX 1050 Ti | Ubuntu Server |
| **Worker Node (ThinkPad)** | `hpc2` | Intel Core i5-6200U | 2C / 4T | Up to 2.80 GHz | 16 GB DDR4 | 500 GB | Integrated Graphics | Ubuntu Server |

---

## 3. Network & System Configuration

### 3.1 Network Topology & Static IPs
Both nodes are connected via local Ethernet with static local IP addresses established to ensure reliable inter-node communication.

* **Master Node (`hpc1`):** `192.168.40.213`
* **Worker Node (`hpc2`):** `192.168.40.83`

### 3.2 Passwordless SSH Connectivity
To facilitate automated cluster orchestration via `mpirun`, passwordless SSH was established between nodes using RSA/ED25519 key pairs.

1. Key generation on primary node:
   ```bash
   ssh-keygen -t ed25519 -C "hpc-cluster-key"
   ```
2. Distributing public key to worker node:
   ```bash
   ssh-copy-id palmer@192.168.40.83
   ```
3. Verification of seamless passwordless entry:
   ```bash
   ssh palmer@192.168.40.83
   ```

---

## 4. Software Environment & Parallel Stack

### 4.1 Dependency Management (PEP 668 Handling)
Modern Ubuntu systems enforce **PEP 668** (`externally-managed-environment`), blocking global system-wide `pip` installs to protect system stability. 

To manage Python-based system utilities safely without interfering with core OS packages, **`pipx`** was installed:
```bash
sudo apt update && sudo apt install -y pipx
pipx ensurepath
source ~/.bashrc
```

### 4.2 Distributed Parallel Processing (`OpenMPI` & `mpi4py`)
The parallel processing layer relies on OpenMPI for low-level C execution and `mpi4py` for Python parallelization.

1. Installed OpenMPI compilers and development headers on both nodes:
   ```bash
   sudo apt install -y openmpi-bin libopenmpi-dev build-essential
   ```
2. Python parallelization binding (`mpi4py`):
   ```bash
   sudo apt install -y python3-mpi4py
   ```

### 4.3 Validation Code Example (High-Load C Benchmark)
The cluster was validated by distributing a massive 50-billion iteration numerical integration (Riemann Sum) calculation for Pi using C with MPI headers:

* **Source File:** `pi_pesado.c`
* **Compilation Command:**
  ```bash
  mpicc -O3 -o pi_pesado pi_pesado.c
  ```
* **Distributed Execution Command:**
  ```bash
  mpirun -H 192.168.40.213,192.168.40.83 -n 2 ./pi_pesado
  ```

---

## 5. Monitoring Infrastructure

### 5.1 CLI Monitoring (Glances)
Installed via `pipx` on both nodes for terminal-based performance inspection:
```bash
pipx install glances
```
* **Local Mode:** `glances`
* **Server Mode (Slave):** `glances -s`
* **Client Mode (Master):** `glances -c 192.168.40.83`

### 5.2 Real-Time Web Monitoring (Netdata)
Deployed Netdata across both nodes to visualize real-time CPU, RAM, disk, and network throughput via a centralized web dashboard:

* **Installation Command (run on both `hpc1` and `hpc2`):**
  ```bash
  wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh --non-interactive
  ```
* **Dashboard URLs:**
  * Master Node: `http://192.168.40.213:19999`
  * Worker Node: `http://192.168.40.83:19999`

---

## 6. Remote Management & Secure Access (Tailscale)

To manage and execute workloads on the HPC cluster from external networks (outside the local home network) without exposing vulnerable ports to the public internet, **Tailscale** (WireGuard mesh VPN) was deployed.

### 6.1 Cluster Nodes Setup (Linux)
1. Installed Tailscale on `hpc1` and `hpc2`:
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   ```
2. Authenticated both nodes to the private network mesh:
   ```bash
   sudo tailscale up
   ```

### 6.2 Client Workstation Setup (Windows)
1. Downloaded and installed Tailscale Windows client.
2. Authenticated with the same user account.

### 6.3 Secure Connection Workflow
With Tailscale active on external clients, secure SSH and Web UI access are achievable via Tailscale IP addresses (`100.x.x.x` range) from anywhere in the world:
```powershell
ssh palmer@100.x.x.x
```

---

## 7. Standard Operating Procedures (SOP)

### 7.1 Power Down Procedure
To prevent file system corruption or stale network locks:
1. Shutdown Worker Node (`hpc2`):
   ```bash
   ssh palmer@192.168.40.83 "sudo shutdown now"
   ```
2. Shutdown Master Node (`hpc1`):
   ```bash
   sudo shutdown now
   ```

### 7.2 Power Up & Verification Procedure
1. Boot both physical machines (`hpc1` and `hpc2`).
2. Test network handshake from Master:
   ```bash
   ping -c 3 192.168.40.83
   ```
3. Netdata and Tailscale services start automatically upon system boot up.
