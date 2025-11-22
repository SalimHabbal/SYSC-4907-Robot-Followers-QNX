# QNX 7.1 for Raspberry Pi 4


This repository contains the build scripts and configuration files to generate a bootable QNX 7.1 disk image for the Raspberry Pi 4. It was developed as part of the **Robot Followers Capstone Project** to create a custom, modular operating system environment.

## 🚀 Overview

The project focuses on creating a lean, efficient, and customized QNX OS image. The build system automates the generation of:
-   **IFS Partition**: Holds the QNX OS image (kernel, startup, drivers).
-   **Root Partition**: A writable QNX filesystem for user data and additional binaries.

The resulting `disk.image` is ready to be flashed to an SD card for the Raspberry Pi 4.

## ✨ Key Features & Implementation

This custom build integrates several critical functionalities to support embedded development and remote telemetry.

### 1. VI Text Editor Integration
To enable on-device file editing without rebuilding the image, the **VI text editor** was integrated directly into the build.
-   **Implementation**: Mapped `/usr/bin/vi` in the `root_partition.build` file.
-   **Benefit**: Allows for quick configuration changes and script editing directly on the RPi4 terminal.

### 2. Wi-Fi Connectivity
Robust wireless networking was implemented to support remote control and future 5G integration.
-   **Startup Sequence**: A custom script (`/etc/init.d/20-wifi`) handles the connection process:
    1.  **Driver Loading**: Loads `devnp-qwdi...so` with firmware and NVRAM settings.
    2.  **Authentication**: Starts `wpa_supplicant` in the background with a 3-second delay to prevent race conditions.
    3.  **DHCP**: Runs `dhclient` only after the interface is up to ensure a valid IP address is obtained.
-   **Configuration**: Uses `wpa_supplicant.conf` for network credentials.

### 3. SSH Remote Access
SSH (Secure Shell) is enabled for headless development and monitoring.
-   **Security**: configured with `RSA` and `ED25519` host keys (read-only permissions).
-   **Access**: `PermitRootLogin` is enabled for development convenience.
-   **Components**: Includes `sshd` (server), `ssh` (client), and `scp` (file transfer).

## 🛠 Prerequisites

Before building, ensure you have the following installed and configured:

-   **QNX Software Development Platform (SDP) 7.1**
-   **QNX Aarch64 Toolchain** (part of SDP 7.1)
-   **Required Utilities** (must be in your PATH):
    -   `mkifs`, `mkqnx6fsimg`, `mkfatfsimg`, `diskimage`
    -   `sed`, `wc`

> [!NOTE]
> Ensure your `QNX_TARGET` and `QNX_HOST` environment variables are correctly set.

## 📂 Project Structure

```text
.
├── build-rpi4-image       # Main build script
└── config/                # Configuration files
    ├── ifs.build          # IFS buildfile (startup, drivers, binaries)
    ├── root_partition.build # Root filesystem buildfile (VI, WiFi script, SSH config)
    ├── disk.layout.TEMPLATE # Template for the disk partition layout
    └── ...
```

## 🔨 Usage

1.  **Clone the repository:**
    ```bash
    git clone <repo_url>
    cd <repo_name>/rpi4-writable-diskimage-build
    ```

2.  **Run the build script:**
    ```bash
    ./build-rpi4-image
    ```
    This generates `output/disk.image`.

3.  **Flash the image (Linux Example):**
    Insert your SD card and identify the device (e.g., `/dev/sda` or `/dev/mmcblk0`).
    > [!WARNING]
    > Be extremely careful with the `dd` command. Writing to the wrong device will destroy data.

    ```bash
    # Replace /dev/sdX with your actual SD card device
    sudo dd if=output/disk.image of=/dev/sdX bs=1M status=progress
    ```

4.  **Boot:**
    Insert the SD card into the RPi4 and power it on. You can connect via Serial (UART) or wait for it to connect to Wi-Fi (if configured) and SSH in.

## 🔌 Hardware Support

| Feature | Driver / Utility | Status |
| :--- | :--- | :--- |
| **UART** | `devc-serminiuart` | ✅ Supported |
| **Wi-Fi** | `devnp-qwdi...` (BCM43455) | ✅ Integrated & Scripted |
| **SSH** | `sshd` (OpenSSH) | ✅ Integrated |
| **Text Editor** | `vi` | ✅ Integrated |
| **Ethernet** | `devnp-genet.so` | ✅ Supported |
| **USB** | `devu-hcd-bcm2711-xhci.so` | ✅ Supported |

---
*Generated for SYSC 4907 Robot Followers Project*
