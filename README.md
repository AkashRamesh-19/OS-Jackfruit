# Jackfruit: Multi-Container Supervised Runtime

## 1. Team Information
* Name: Akash Ramesh (PES1UG24AM212)
* Name: Prithvi Warrier (PES1UG24AM205)

---

## 2. Comprehensive Setup & Execution

### 2.1 Environment Preparation
sudo apt update && sudo apt install -y linux-headers-$(uname -r)

chmod +x environment-check.sh
sudo ./environment-check.sh

---

### 2.2 Root Filesystem Setup
mkdir -p rootfs-base rootfs-alpha rootfs-beta

wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz

tar -xzf alpine-minirootfs-*.tar.gz -C rootfs-base

cp -a ./rootfs-base/. ./rootfs-alpha/
cp -a ./rootfs-base/. ./rootfs-beta/

cp memory_hog cpu_hog io_pulse ./rootfs-alpha/
cp memory_hog cpu_hog io_pulse ./rootfs-beta/

---

### 2.3 Build and Load
cd boilerplate

make clean
make

sudo rmmod monitor 2>/dev/null
sudo insmod monitor.ko

ls -l /dev/container_monitor

---

## 3. Demo Execution

### 3.1 Start Supervisor
sudo ./engine supervisor ../rootfs-base

---

### 3.2 Run Containers

Simple Execution
sudo ./engine run alpha ../rootfs-alpha /bin/echo HELLO

PID Isolation
sudo ./engine run alpha ../rootfs-alpha /bin/ps

Filesystem Isolation
sudo ./engine run alpha ../rootfs-alpha /bin/ls -F /

Memory Monitoring
sudo ./engine run alpha ../rootfs-alpha ./memory_hog --soft-mib 10 --hard-mib 50

Multiple Containers
sudo ./engine run alpha ../rootfs-alpha /bin/sleep 5
sudo ./engine run beta ../rootfs-beta /bin/sleep 5

---

### 3.3 Kernel Logs
sudo dmesg | tail -n 10

---

## 4. Engineering Analysis

### 4.1 Isolation Logic
We implemented container isolation using Linux namespaces:

- PID Namespace: Each container has its own process tree
- UTS Namespace: Separate hostname per container
- Mount Namespace: Filesystem isolated using chroot()
- /proc Mount: Ensures correct process visibility

---

### 4.2 IPC and Logging

- Control Plane: Unix Domain Sockets (AF_UNIX)
- Logging:
  - Producer: container stdout/stderr
  - Consumer: logging thread
  - Buffer: bounded buffer

This prevents blocking and ensures efficient logging.

---

### 4.3 Kernel Module Monitoring

- Tracks Resident Set Size (RSS)
- Maintains list of container PIDs
- Periodically checks memory usage
- Enforces:
  - Soft limit → warning
  - Hard limit → SIGKILL

---

## 5. Design Tradeoffs

- Polling vs Event-Based:
  We use periodic polling (1 second) for simplicity.

- Static Binaries:
  Ensures execution inside Alpine without dependencies.

- Command Execution:
  Uses /bin/sh -c to support arguments like:
  sleep 5
  ls -F /

---

## 6. Features Implemented

- Container creation using clone()
- Namespace isolation (PID, UTS, Mount)
- Filesystem isolation using chroot()
- Kernel memory monitoring
- Logging using bounded buffer
- Multi-container support
- IPC using Unix sockets
- Command execution with arguments

---

## 7. Conclusion

This project demonstrates a lightweight container runtime with kernel-level monitoring. It combines OS concepts such as process isolation, memory control, IPC, and synchronization into a working system.
