# Hitachi Storage Plug-in for Proxmox VE v1.0.0

The **Hitachi Storage Plug-in for Proxmox VE** integrates Proxmox Virtual Environment (VE) with Hitachi Virtual Storage Platform One Block (VSP One Block) storage systems. It enables Proxmox VE administrators to provision and manage Hitachi storage volumes for virtual machines (VMs) and containers (CTs) directly from the Proxmox VE environment.

## Table of Contents

- [Overview](#overview)
- [Supported Features](#supported-features)
- [Hardware Requirements](#hardware-requirements)
- [Software Requirements](#software-requirements)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [Known Issues and Limitations](#known-issues-and-limitations)
- [Documentation](#documentation)
- [Support](#support)

## Overview

The Storage Plug-in for Proxmox VE integrates existing Hitachi storage pools with Proxmox VE. The plug-in uses the storage system REST API for storage management and Fibre Channel (FC) or iSCSI for host connectivity. It automatically maps and unmaps storage volumes to Proxmox VE hosts as required by workload operations and supports multi-port and multipath configurations.

Each Proxmox VE image provisioned through the plug-in corresponds to a volume on the Hitachi storage system.

## Supported Features

| Feature | Description |
|---|---|
| Storage pool management | Add, update, remove, enable, and disable Hitachi storage pools. |
| Multiple storage systems and pools | Configure multiple supported storage systems and storage pools within a Proxmox VE environment. |
| Shared storage | Share Hitachi storage pools across Proxmox VE nodes. |
| Volume lifecycle management | Create, attach, detach, resize (increase capacity only), and delete volumes. |
| Automated storage mapping | Map and unmap volumes to Proxmox VE hosts during workload operations. |
| VM and container storage | Provision storage for VMs and CTs using standard Proxmox VE workflows. |
| Volume cloning | Support full-clone and linked-clone workflows for VMs and CTs. |
| Backup and restore | Back up and restore VMs using Proxmox VE workflows. |
| Migration | Support shared-storage VM migration (live and offline), cross-pool migration, and cross-array migration. |
| Data Reduction Sharing (DRS) | When DRS is configured on the storage system, provision images as vClones of a parent volume with deduplication and compression enabled. |

Refer to the Installation and User Guide for the complete list of supported operations.

## Hardware Requirements

| Supported storage systems | Minimum microcode version | Interface |
|---|---|---|
| VSP One Block 24 | A3-05-42-40/xx | Fibre Channel, iSCSI |
| VSP One Block 26 | A3-05-42-40/xx | Fibre Channel, iSCSI |
| VSP One Block 28 | A3-05-42-40/xx | Fibre Channel, iSCSI |
| VSP One Block 85 | A0-05-21-00/xx | Fibre Channel, iSCSI |

For the latest supported storage models and microcode versions, refer to the product release notes.

## Software Requirements

| Software | Supported version |
|---|---|
| Proxmox VE (Debian OS 13.x) | 9.2.x or later |
| Storage Plug-in for Proxmox VE installer package | 1.0.0.x |
| `multipathd` | 0.11.1-2 or later |
| `udevadm` | 257 or later |
| `iscsiadm` | 2.1.11 or later |
| `dmsetup` | 1.02.205 (2025-02-27) or later |

The `udevadm`, `iscsiadm`, and `dmsetup` utilities are available in the Proxmox VE environment and do not require additional installation.

## Installation

### Prerequisites

- A supported Proxmox VE environment with GUI and SSH access.
- Administrative privileges on the Proxmox VE nodes.
- A supported Hitachi storage system with an existing storage pool and storage administrator access.
- FC or iSCSI connectivity between the Proxmox VE nodes and the storage system.
- A CA certificate for SSL certificate validation.

Review the system requirements and storage port and LDEV configuration requirements in the Installation and User Guide before installing the plug-in.

### Download and install the plug-in

Download the Debian installation package (`.deb`) from the **Assets** section of the [latest GitHub release](https://github.com/hitachi-vantara/Hitachi-Storage-Plugin-for-Proxmox-VE/releases/latest).

For a specific version, see [all releases](https://github.com/hitachi-vantara/Hitachi-Storage-Plugin-for-Proxmox-VE/releases).

Install the plug-in on each Proxmox VE node that requires access to Hitachi storage.

1. Log in to the Proxmox VE node as root and install the multipath utility if needed:

   ```bash
   apt install multipath-tools
   ```

2. Navigate to the directory containing the downloaded package and install it. Replace `<version>` with the version in the downloaded filename.

   ```bash
   apt install ./libpve-storage-hitachi-hspv-plugin_<version>.deb
   ```

3. When prompted, enter the path to the CA certificate file, or configure it later in `hspv.json` before connecting to the storage system.

4. Verify the installed plug-in version:

   ```bash
   hspvver
   ```

For detailed installation and uninstallation instructions, see the [Installation and User Guide](https://docs.hitachivantara.com/v/u/en-us/mk-26adptr166/latest).

## Getting Started

### Configure the plug-in

The primary plug-in configuration file is located at `/etc/pve/priv/hspv/hspv.json`. SSL validation is enabled by default. Configure a valid CA certificate in the `ssl_ca_file` setting before connecting to the storage system.

After modifying `hspv.json`, reload or restart the Proxmox VE services:

```bash
systemctl reload-or-restart pveproxy pvedaemon pvestatd
```

Do not modify other files in `/etc/pve/priv/hspv` or manually change the plug-in storage pool names in `/etc/pve/storage.cfg`.

### Add a Hitachi storage pool

Use the Proxmox VE CLI to register an existing Hitachi storage pool with the plug-in. The storage type is `hitachi_hspv`.

```bash
pvesm add hitachi_hspv hitachi_pool1 \
  --hspv_address '<storage-address>' \
  --hspv_pool '<pool-name-or-number>' \
  --hspv_ports '<port-list>' \
  --hspv_username '<username>' \
  --hspv_password '<password>' \
  --hspv_ldev_range '100-200'
```

Replace the placeholders with the values for your environment. The LDEV range is optional; the range shown is an example. For a Proxmox VE cluster, specify the `--shared` setting (`1` for shared storage or `0` for non-shared storage). Review the guide for port group and LDEV range requirements. Protect storage credentials according to your organization's security practices.

Verify the storage pool status:

```bash
pvesm status
```

### Provision storage for a VM or container

In the Proxmox VE GUI, select the Hitachi storage pool when creating a VM or CT, or when adding a disk to an existing workload. The plug-in creates the corresponding volume on the Hitachi storage system.

For example, to allocate a 1 GiB image for an existing VM with ID `333` using the CLI:

```bash
pvesm alloc hitachi_pool1 333 '' 1G
pvesm list hitachi_pool1
```

The filename argument must be an empty string because custom image names are not supported in this release.

For additional configuration and storage management procedures, refer to the Installation and User Guide.

## Known Issues and Limitations

Version 1.0.0 is the initial release of the Storage Plug-in for Proxmox VE. The following limitations apply:

- Adding and updating Hitachi storage pools is supported through the CLI only (`pvesm add` and `pvesm set`).
- Provisioned volumes support only the `raw` image format.
- Volume capacity can be increased but not decreased.
- Hibernation is not supported for VMs with disks provisioned from Hitachi storage pools.
- Only one SSL certificate file is supported per Proxmox VE node.
- Full clones copy data through the Proxmox VE host and may take longer for large volumes.
- Large-volume deletion with Adaptive Data Reduction (ADR) enabled may exceed the Proxmox VE timeout and leave a volume on the storage system.
- A vClone parent volume may remain after its child volumes are deleted.

Refer to the release notes for the complete list of known issues, limitations, and applicable workarounds.

## Documentation

For detailed installation, configuration, supported features, and usage instructions, refer to the official product documentation:

| Document | Document number | Link |
|---|---|---|
| Hitachi Storage Plug-in for Proxmox VE v1.0.0 Installation and User Guide | MK-26ADPTR166-00 | [Installation and User Guide](https://docs.hitachivantara.com/v/u/en-us/mk-26adptr166/latest) |
| Hitachi Storage Plug-in for Proxmox VE v1.0.0 Release Notes | RN-26ADPTR162-00 | [Hitachi Vantara Documentation](https://docs.hitachivantara.com/v/u/en-us/rn-26adptr162/latest) |

Additional resources:

- [Hitachi Vantara Documentation](https://docs.hitachivantara.com/)
- [Proxmox VE Documentation](https://pve.proxmox.com/pve-docs/)

Review the release notes before installing or using the plug-in.

## Support

For product-related questions, installation assistance, or technical issues, use the following support channels:

- [Hitachi Vantara Support Portal](https://support.hitachivantara.com/)
- [Contact Hitachi Vantara Support](https://support.hitachivantara.com/en/contact-support.html)
- [Hitachi Vantara Community](https://community.hitachivantara.com/s/)

For documentation feedback, contact [doc.feedback@hitachivantara.com](mailto:doc.feedback@hitachivantara.com).

---

Copyright © 2026 Hitachi Vantara LLC. All rights reserved.

This project is licensed under Apache 2.0
