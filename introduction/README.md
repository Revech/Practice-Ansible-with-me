# Ansible Practice Lab Documentation

## 1. Introduction

This documentation provides a structured guide for building an **Ansible practice lab** on a Fedora-based workstation.  
It captures the configuration details, errors, and troubleshooting encountered during the setup process.  

The purpose of this lab is to establish a controlled environment for experimenting with Ansible automation, simulating real-world infrastructure management tasks through virtualized hosts.  

---

## 2. Environment Overview

### 2.1 Operating System
- Fedora Workstation 42 (Linux Kernel 6.16.3-200)  

### 2.2 Hardware Platform
- **Model**: HP ZBook Mobile Workstation  
- **Processor**: Intel Core i9   
- **Memory**: 32 GB DDR4 RAM  
- **Storage**: 512 GB NVMe SSD  
- **Graphics**: NVIDIA RTX 2000 Ada Generation  

### 2.3 Virtualization Stack
- **Primary**:  
  - KVM (Kernel-based Virtual Machine)  
  - QEMU (Quick Emulator)  
  - libvirt (Management layer for VMs)  

- **Secondary (Attempted)**:  
  - VMware Workstation (failed due to kernel module incompatibility)  

---

## 3. VMware Workstation Compatibility Issue

### 3.1 What Happened
VMware Workstation was downloaded and installed, but the required kernel modules (`vmmon`, `vmnet`) failed to build against Fedora’s **6.16.3-200 kernel**.  
The build process stopped with errors such as:  
- **Macro redefinition**: `MAX` already defined in kernel headers (`vnetInt.h`)  
- **Deprecated init function**: `objtool: init_module() ... deprecated`  

These issues occur because VMware’s kernel drivers are **out-of-tree** (not shipped with the Linux kernel) and often lag behind Fedora’s rapid kernel updates.  

---

### 3.2 Background Concepts

- **Linux Kernel**  
  The kernel is the “engine” of the OS. It manages CPU, memory, devices, and provides APIs for drivers and applications.  
  Kernel APIs change frequently across releases (e.g., 6.14 → 6.16), unlike stable user-space libraries.  

- **Kernel Modules**  
  VMware Workstation requires two modules to function:  
  - `vmmon` → virtualization hooks for guest execution.  
  - `vmnet` → virtual networking between host, guest, and external networks.  

  Without these modules, VMware cannot start VMs.  

---

### 3.3 Why Compatibility Breaks
- VMware’s drivers are **not part of the Linux kernel**.  
- Fedora introduced changes in kernel 6.15/6.16:  
  - New definitions (`MAX`) conflict with VMware’s code.  
  - Old functions (`init_module()`) were removed.  
- VMware has not yet updated its modules, leading to build errors.  

---

### 3.4 Workarounds
1. **Community Patches**  
   - Projects like [mkubecek/vmware-host-modules](https://github.com/mkubecek/vmware-host-modules) maintain patch sets that adjust VMware’s source code to align with the latest kernel.  
   - These patches act as translators between VMware’s outdated module code and new kernel APIs.  

2. **Boot Older Kernel**  
   - Fedora keeps older kernels installed. Booting into a 6.14 kernel allows VMware modules to build successfully until official patches are released.  

---

### 3.5 What I Chose
- VMware on Fedora will often break after kernel updates until patched.  
- Alternatives like **KVM/libvirt** avoid this problem because their virtualization drivers are **in-tree** (shipped with the kernel and updated in sync).  
- For this lab, **KVM/QEMU/libvirt** is the more stable and recommended option on Fedora.  

---

## 4. System Verification Commands

For reproducibility, the following commands can be used to verify system specifications:

```bash
# CPU details
lscpu

# Memory details
free -h

# Storage devices
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT

# GPU details
lspci | grep -i vga

# OS version
cat /etc/fedora-release

