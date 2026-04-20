# Jackfruit: Multi-Container Supervised Runtime

## 1. Team Information
* **Name:** Akash Ramesh (PES1UG24AM212)
* **Name:** Prithvi Warrier (PES1UG24AM205)

---

## 2. Comprehensive Setup & Execution

### 2.1 Environment Preparation

```bash
sudo apt update && sudo apt install -y linux-headers-$(uname -r)
chmod +x environment-check.sh
sudo ./environment-check.sh
```

---

### 2.2 Root Filesystem Setup

```bash
mkdir -p rootfs-base rootfs-alpha rootfs-beta
wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz
tar -xzf alpine-minirootfs-*.tar.gz -C rootfs-base
cp -a ./rootfs-base/. ./rootfs-alpha/
cp -a ./rootfs-base/. ./rootfs-beta/
cp memory_hog cpu_hog io_pulse ./rootfs-alpha/
cp memory_hog cpu_hog io_pulse ./rootfs-beta/
```

---

### 2.3 Build and Load

```bash
cd boilerplate
make clean
make
sudo rmmod monitor 2>/dev/null
sudo insmod monitor.ko
ls -l /dev/container_monitor
```

---

## 3. Demo Execution

### 3.1 Start Supervisor

```bash
sudo ./engine supervisor ../rootfs-base
```

---

### 3.2 Run Containers

**Simple Execution**

```bash
sudo ./engine run alpha ../rootfs-alpha /bin/echo HELLO
```

**PID Isolation**

```bash
sudo ./engine run alpha ../rootfs-alpha /bin/ps
```

**Filesystem Isolation**

```bash
sudo ./engine run alpha ../rootfs-alpha /bin/ls -F /
```

**Memory Monitoring**

```bash
sudo ./engine run alpha ../rootfs-alpha ./memory_hog --soft-mib 10 --hard-mib 50
```

**Multiple Containers**

```bash
sudo ./engine run alpha ../rootfs-alpha /bin/sleep 5
sudo ./engine run beta ../rootfs-beta /bin/sleep 5
```

---

### 3.3 Kernel Logs

```bash
sudo dmesg | tail -n 10
```

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
- Logging system:
  - Producer: container stdout/stderr  
  - Consumer: logging thread  
  - Buffer: bounded buffer  

---

### 4.3 Kernel Module Monitoring

- Tracks Resident Set Size (RSS)  
- Maintains container PID list  
- Periodically checks memory  
- Enforces:
  - Soft limit → warning  
  - Hard limit → SIGKILL  

---

## 5. Design Tradeoffs

- Polling used instead of event-based monitoring (simpler, low overhead)  
- Static binaries ensure compatibility inside Alpine  
- `/bin/sh -c` used for flexible command execution  

---

## 6. Features Implemented

- Container creation using clone()  
- Namespace isolation (PID, UTS, Mount)  
- Filesystem isolation using chroot()  
- Kernel memory monitoring  
- Logging with bounded buffer  
- Multi-container support  
- IPC via Unix sockets  
- Command execution with arguments  

---

## 7. Conclusion

This project demonstrates a lightweight container runtime integrating process isolation, memory control, IPC, and synchronization into a functional system.
