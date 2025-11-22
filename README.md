# QNX Neutrino 7.1 for Raspberry Pi 4

![QNX Logo](https://www.qnx.com/content/dam/qnx/images/logos/qnx-logo-black.svg) <!-- Replace with a local asset if available, or keep generic -->

This repository contains the build scripts and configuration files to generate a bootable QNX 7.1 disk image for the Raspberry Pi 4. It is designed to create a custom IFS (Image Filesystem) and a writable root filesystem.

## 🚀 Overview

The build process automates the creation of a disk image containing:
-   **IFS Partition**: Holds the QNX OS image (kernel, startup, procnto, drivers).
-   **Root Partition**: A writable QNX6 filesystem for user data and additional binaries.

The resulting `disk.image` can be flashed directly to an SD card.

## 🛠 Prerequisites

Before building, ensure you have the following installed and configured:

-   **QNX Software Development Platform (SDP) 7.1**
-   **QNX Aarch64 Toolchain** (part of SDP 7.1)
-   **Required Utilities** (must be in your PATH):
    -   `mkifs`: To build the Image Filesystem.
    -   `mkqnx6fsimg`: To create the QNX6 root filesystem image.
    -   `mkfatfsimg`: To create the FAT partition image.
    -   `diskimage`: To assemble the final disk image.
    -   `sed`, `wc`: Standard Unix utilities.

> [!NOTE]
> Ensure your `QNX_TARGET` and `QNX_HOST` environment variables are correctly set.

## 📂 Project Structure

```text
.
├── build-rpi4-image       # Main build script
└── config/                # Configuration files
    ├── ifs.build          # IFS buildfile (startup, drivers, binaries)
    ├── root_partition.build # Root filesystem buildfile
    ├── disk.layout.TEMPLATE # Template for the disk partition layout
    ├── ifs_partition.build # FAT partition config for IFS
    └── ...
```

## ⚙️ Configuration

### IFS Build (`config/ifs.build`)
Defines the contents of the bootable OS image. Key sections include:
-   **Startup**: `startup-bcm2711-rpi4` with arguments for debug console (`miniuart` or `pl011`).
-   **Drivers**: Serial, I2C, SPI, SDMMC, Network (Genet), USB (XHCI), Audio.
-   **Binaries**: Common utilities (`ls`, `cp`, `pidin`, etc.) and libraries.
-   **Scripts**: `.script` section defines the boot sequence (starting services, drivers, mounting filesystems).

### Root Partition (`config/root_partition.build`)
Defines the structure of the writable root filesystem.
-   **Directories**: Creates standard hierarchy (`/etc`, `/home`, `/var`, etc.).
-   **Files**: Includes configuration files (WiFi, SSH) and additional binaries not in the IFS.

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

    The script will:
    -   Generate the root partition image (`output/root.part`).
    -   Generate the IFS image (`output/ifs.bin`) and partition (`output/ifs.part`).
    -   Calculate the required disk size.
    -   Create the final disk image (`output/disk.image`).

3.  **Flash the image:**
    Use `dd` or a tool like Etcher to write `output/disk.image` to your SD card.
    ```bash
    # Example (BE CAREFUL with the device path!)
    sudo dd if=output/disk.image of=/dev/sdX bs=4M status=progress
    ```

## 🔌 Hardware Support

| Feature | Driver / Utility | Status |
| :--- | :--- | :--- |
| **UART** | `devc-serminiuart`, `devc-serpl011` | ✅ Supported |
| **I2C** | `i2c-bcm2711` | ✅ Supported |
| **SPI** | `spi-master` | ✅ Supported |
| **SD Card** | `devb-sdmmc-bcm2711` | ✅ Supported |
| **Ethernet** | `devnp-genet.so` | ✅ Supported |
| **WiFi** | `devnp-qwdi...` (BCM43455) | ✅ Configured |
| **USB** | `devu-hcd-bcm2711-xhci.so` | ✅ Supported |
| **Audio** | `io-audio` (PWM / PCM512x) | ✅ Supported |
| **GPIO** | `gpio-bcm2711` | ✅ Supported |

## 📝 Customization

-   **Change Debug Console**: Edit `config/ifs.build` to switch between `miniuart` (default) and `pl011`.
-   **Add Files**: Add binaries or files to `config/ifs.build` (for read-only fast access) or `config/root_partition.build` (for writable storage).
-   **WiFi Config**: Edit `files/wifi/wpa_supplicant.conf` (referenced in build files) to set your network credentials.

---
*Generated for SYSC 4907 Robot Followers Project*
