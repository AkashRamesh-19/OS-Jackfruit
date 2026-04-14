# Multi-Container Runtime (OS Project)

## Overview

This project implements a lightweight Linux container runtime in C. It supports running multiple isolated containers using chroot and process isolation.

## Features

* Multi-container execution
* Supervisor-based container management
* Process isolation using namespaces
* Logging system for containers
* Support for CPU, memory, and I/O workloads

## How to Run

### Build

```
cd boilerplate
make
```

### Run Single Container

```
sudo ./engine run c1 ../rootfs-alpha "/bin/sh"
```

### Run Multiple Containers

```
sudo ./engine start c1 ../rootfs-alpha "/bin/sh"
sudo ./engine start c2 ../rootfs-beta "/bin/sh"
```

## Example Output

* Container runs isolated shell
* `hostname` shows container ID
* `ps` shows isolated processes

## Technologies Used

* C Programming
* Linux System Calls
* chroot
* Process Management

## Conclusion

The project demonstrates how container runtimes work internally by managing isolated environments and processes.

---
