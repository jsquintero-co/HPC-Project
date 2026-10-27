# HPC Cluster Technical Specification 

**Project:** High-Performance & Distributed Computing Infrastructure

**Workload Manager:** Slurm Workload Manager

**Cluster Architecture:** Heterogeneous (CPU/GPU-Enabled Master Node)

**Status:** Operational

## 1. System Architecture & Network Topology

* **Orchestration & Scheduling:** Slurm Workload Manager (`slurmctld` on `hpc1`, `slurmd` on `hpc1` and `hpc2`).

* **Distributed File System:** Centralized NFS (*Network File System*) mount point for synchronized access to scripts, binaries, and datasets across all nodes.

* **Inter-Node Authentication:** MUNGE (*MUNGE Uid N' Gid Emporium*) for secure credential signing and verification.

* **Networking:** Dedicated Ethernet Local Area Network (LAN) with open Slurm control ports (`6817/TCP`) and task execution ports (`6818/TCP`) for bidirectional communication.

## 2. Hardware Specifications per Node

### Master / Compute Node (`hpc1`)

* **Role:** Controller & Worker Node (`slurmctld` / `slurmd`)

* **Processor (CPU):** Intel Core i5-10400F @ 4.30 GHz Max Turbo (6 Cores / 12 Threads)

* **System Memory (RAM):** 24 GB DDR4

* **Graphics Acceleration (GPU):** NVIDIA GeForce GTX 1050 Ti (4 GB VRAM)

* **Storage:** 700 GB SSD (Host for NFS mount point)

* **Operating System:** Ubuntu Linux

### Worker Node (`hpc2`)

* **Role:** Dedicated Worker Node (`slurmd`)

* **Processor (CPU):** Intel Core i5-6200U @ 2.80 GHz Max Turbo (2 Cores / 4 Threads)

* **System Memory (RAM):** 16 GB DDR4

* **Graphics Acceleration (GPU):** N/A

* **Storage:** 500 GB (NFS client mounted at `/hpc_share`)

* **Operating System:** Ubuntu Linux

## 3. Global Cluster Resource Summary

| Parameter | Total Cluster Capacity | 
| ----- | ----- | 
| **Total Nodes** | 2 Compute Nodes (`hpc1`, `hpc2`) | 
| **Total Logical Threads (vCPU)** | 16 Processing Threads | 
| **Total RAM** | 40 GB DDR4 | 
| **Total Storage** | 1.2 TB (700 GB Host / 500 GB Worker) | 
| **Hardware Acceleration** | 1x NVIDIA GTX 1050 Ti (Locally available on `hpc1`) | 
| **Execution Mode** | Asynchronous / Batch (`sbatch`) and Parallel execution (`srun`) | 

## 4. Workload Profile & Software Compatibility

### Prototyping and Execution Capabilities

* **CPU Distributed Computing:** Optimized for compute-intensive workloads, stochastic simulations (Monte Carlo), parallel C/C++ applications, and multi-process distributed scripts (16 combined threads).

* **Classic Machine Learning (Multi-Node CPU):** Configured for CPU-based parallel workloads (Scikit-Learn, XGBoost, Random Forest, parallel cross-validation, and hyperparameter grid search leveraging 40 GB combined RAM).

* **GPU Acceleration (Single-Node / `hpc1`):** Supported for local prototyping and light Deep Learning training via CUDA/PyTorch/TensorFlow on `hpc1`.

* **Data Processing:** Compatible with in-memory distributed datasets via NFS (Pandas, Dask, Python multiprocessing).

### Available Environment & Tooling

* **Compilers:** GCC / G++ with advanced optimization support (`-O3`).

* **Runtimes & Libraries:** Python 3.x, Bash Shell, NVIDIA CUDA Drivers (on `hpc1`).

* **Workflow Automation:** Full support for asynchronous batch processing via `#SBATCH` directives and standardized logging outputs.
