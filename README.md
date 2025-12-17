# G1OS
Our aim was to provide an effective and efficient Operating system.
We are developing an operating system that consists of a kernel with multiprocessing features to help improve the computing capacity, as well as the storage capacity of the System 
Cloud-based operating systems provide a more secure computing environment than traditional on-premise systems. Cloud providers often have teams of security experts and implement best practices for security measures to protect against cyber threats.
Cloud-based operating systems provide more flexibility and remote work capabilities, which are particularly relevant in today's world, where more and more people are working remotely. With cloud-based operating systems, users can access their desktop environments and files from anywhere with an internet connection, which allows for more collaboration and productivity.

*Objective:
To provide fast execution of system processes and programs. Establishes communication between computer hardware (i.e. CPU, disk memory) and user application.
Accessibility
Scalability
Security
Reliability
Collaboration
Efficiency

*Proposed Architecture
<img width="1426" height="706" alt="image" src="https://github.com/user-attachments/assets/3ff42bac-e870-4128-8770-a77dce23accf" />

*Algorithm
<img width="1118" height="595" alt="image" src="https://github.com/user-attachments/assets/3e5d0022-b9e9-4f98-8bab-adcb77ee0917" />

*After Creation

<img width="789" height="549" alt="image" src="https://github.com/user-attachments/assets/dc83a582-7907-497e-a051-2a891a829e8e" />

*Hosting on Cloud
1. Login to AWS account
2. Choose EC2 and Launch Instance
3. Choose Linux based O.S from the available AMI
4. Choose Instance Type
5. Configure the Instance
6. Review and edit Storage settings
7. Add Tags
8. Configure Security and Launch
9. Connect to instance using SSH
10. Once connected, you can install any additional software and configure the system as needed

Performance Evaluation:
In evaluating the performance of our operating system, we have focused on the effectiveness of the monolithic kernel. We have tested the kernel in various controlled environments to determine its efficiency and reliability. Our team has also focused on addressing any errors that have been identified during the testing process.
Overall, our evaluation of the monolithic kernel in our operating system has demonstrated its critical importance to the system's overall performance. While there are challenges in detecting errors, our team is committed to improving this aspect of the kernel's functionality to ensure optimal performance of our operating system.


HOW I DID IT
# G1OS – Execution & Build Guide

G1OS is a **minimal, cloud-oriented Linux operating system** built from source using a custom build pipeline. The system produces a **bootable ISO and disk image**, which can be tested using QEMU or deployed on cloud infrastructure such as AWS EC2.

---

## 1. Prerequisites

Before building G1OS, ensure the host system meets the following requirements:

### Host System

* Linux-based host (Ubuntu/Debian recommended)
* At least 20 GB free disk space
* Minimum 8 GB RAM (16 GB recommended)

### Required Packages

```bash
sudo apt update
sudo apt install -y \
  build-essential \
  gcc g++ make \
  bison flex \
  libssl-dev \
  libelf-dev \
  bc \
  wget curl \
  xz-utils \
  cpio \
  rsync \
  qemu-system \
  dosfstools mtools \
  syslinux isolinux
```

---

## 2. Project Structure Overview

G1OS follows a **staged build pipeline**, where each stage performs a specific task and passes its output to the next stage.

High-level structure:

```text
G1OS/
├── build.sh                # Main entry point
├── scripts/                # All automation scripts
├── config/                 # Kernel, BusyBox, boot configs
├── rootfs/                 # Root filesystem & overlay
├── boot/                   # Bootloader artifacts
├── tools/                  # Testing & deployment utilities
└── docs/                   # Architecture & documentation
```

---

## 3. Execution Entry Point

### Main Build Command

To build the complete operating system, run:

```bash
./build.sh
```

This script orchestrates the **entire build pipeline**, executing each stage in sequence. Individual stages can also be run manually if needed.

---

## 4. Build Pipeline Breakdown

The build process is divided into **logical chunks**, each responsible for a core OS component.

---

### Stage 1: Environment Cleanup

**Script:** `00_clean.sh`

Purpose:

* Removes previous build artifacts
* Resets working directories
* Ensures a clean and reproducible build

This step prevents conflicts caused by stale files or partial builds.

---

### Stage 2: Kernel Acquisition & Compilation

**Scripts:**

* `01_get_kernel.sh`
* `02_build_kernel.sh`

Purpose:

* Downloads the Linux kernel source
* Applies custom configuration
* Compiles the kernel with multiprocessing support

Output:

* Kernel image (`bzImage` or `vmlinuz`)
* Kernel modules

---

### Stage 3: C Library (glibc)

**Scripts:**

* `03_get_glibc.sh`
* `04_build_glibc.sh`

Purpose:

* Fetches the GNU C Library
* Builds glibc for the target system

glibc provides the **core runtime environment** required by user-space applications.

---

### Stage 4: Sysroot Preparation

**Script:** `05_prepare_sysroot.sh`

Purpose:

* Creates an isolated sysroot
* Combines kernel headers, glibc, and toolchain artifacts

The sysroot ensures consistent linking and runtime behavior.

---

### Stage 5: BusyBox Build

**Scripts:**

* `06_get_busybox.sh`
* `07_build_busybox.sh`

Purpose:

* Downloads BusyBox source
* Builds a minimal set of UNIX utilities

BusyBox provides essential commands such as `sh`, `ls`, `mount`, and `init`.

---

### Stage 6: Bundle & Root Filesystem Generation

**Scripts:**

* `08_prepare_bundles.sh`
* `09_generate_rootfs.sh`

Purpose:

* Organizes binaries and libraries into bundles
* Constructs the base root filesystem hierarchy

Output:

* `/bin`, `/sbin`, `/etc`, `/usr`, `/lib`

---

### Stage 7: Initramfs Packaging

**Script:** `10_pack_rootfs.sh`

Purpose:

* Compresses the root filesystem
* Generates an initramfs image

This image is loaded by the kernel at boot time.

---

### Stage 8: Overlay Filesystem Generation

**Script:** `11_generate_overlay.sh`

Purpose:

* Applies custom overrides
* Injects additional configuration and scripts

The overlay allows customization without rebuilding the base system.

---

### Stage 9: Bootloader Setup

**Scripts:**

* `12_get_syslinux.sh` (BIOS)
* `12_get_systemd-boot.sh` (UEFI)

Purpose:

* Installs and configures bootloaders
* Enables both legacy BIOS and modern UEFI boot

---

### Stage 10: ISO Preparation & Generation

**Scripts:**

* `13_prepare_iso.sh`
* `14_generate_iso.sh`

Purpose:

* Creates ISO directory layout
* Generates a bootable ISO image

Output:

* `g1os.iso`

---

### Stage 11: Disk Image Generation

**Script:** `15_generate_image.sh`

Purpose:

* Generates a bootable HDD/USB image
* Suitable for virtual machines or physical media

---

### Stage 12: Cleanup

**Script:** `16_cleanup.sh`

Purpose:

* Removes temporary files
* Reduces disk usage
* Leaves only final artifacts

---

## 5. Testing & Execution

### Run Using QEMU (BIOS)

```bash
tools/testing/qemu-bios.sh
```

### Run Using QEMU (UEFI)

```bash
tools/testing/qemu-uefi.sh
```

These scripts boot the generated image in a virtual environment for validation.

---

## 6. Cloud Deployment (AWS EC2)

G1OS can be deployed in a cloud environment using an EC2 instance:

1. Launch an EC2 instance using a Linux-based AMI
2. Upload or attach the generated disk image
3. Connect via SSH
4. Run and configure G1OS as required

This enables remote access, scalability, and collaboration.

---

## 7. Summary

G1OS uses a **clean, staged execution model** to build a minimal Linux operating system from source. Each stage focuses on a single responsibility, making the system modular, maintainable, and easy to extend.

This project demonstrates strong knowledge of:

* Linux internals
* Kernel and libc compilation
* Filesystem design
* Bootloader configuration
* Virtualization and cloud deployment

---

## 8. License

This project is licensed under the MIT License.







