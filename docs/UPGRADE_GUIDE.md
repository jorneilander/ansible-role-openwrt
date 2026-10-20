# OpenWRT Device Upgrade Guide

This guide explains how to upgrade your OpenWRT device using Ansible, what configuration options affect which features,
and how the system behaves based on those settings.

## Table of Contents

1. [Quick Start](#quick-start)
2. [Prerequisites](#prerequisites)
3. [Configuration Options](#configuration-options)
4. [Upgrade Process Flow](#upgrade-process-flow)
5. [Feature Behavior Matrix](#feature-behavior-matrix)
6. [Best Practices](#best-practices)

---

## Quick Start

```yaml
- hosts: openwrt_devices
  gather_facts: true
  tasks:
    - name: Upgrade OpenWRT device
      ansible.builtin.import_role:
        name: openwrt
      vars:
        openwrt_id: "your_device_id" # Required: Device identifier
        openwrt_platform: "your_platform" # Required: Hardware platform
        openwrt_version: "latest" # Optional: Version to install
```

---

## Prerequisites

Before upgrading, ensure you have:

1. **Ansible installed** on your control machine (v2.18+)
2. **Device access**: SSH credentials or Ansible inventory configured
3. **Network connectivity** between control machine and target device
4. **Sufficient storage** on the device for firmware download (~50-100MB)

---

## Configuration Options

### Core Upgrade Settings

| Variable           | Type   | Default    | Description                                                                           | Impact                                         |
| ------------------ | ------ | ---------- | ------------------------------------------------------------------------------------- | ---------------------------------------------- |
| `openwrt_id`       | string | _required_ | Device identifier from OpenWRT Firmware Selector                                      | **Critical** - Identifies exact hardware model |
| `openwrt_platform` | string | _required_ | Hardware platform (e.g., `ramips/mt7621`)                                             | **Critical** - Determines compatible firmware  |
| `openwrt_version`  | string | `"latest"` | OpenWRT version to install (`"SNAPSHOT"`, `"release_candidate"`, or specific version) | Controls which firmware is installed           |

### Upgrade Behavior Settings

| Variable                                  | Type    | Default | Description                                                 | Impact                                                   |
| ----------------------------------------- | ------- | ------- | ----------------------------------------------------------- | -------------------------------------------------------- |
| `openwrt_version_upgrade_enabled`         | boolean | `true`  | Enable/disable automatic OpenWRT version upgrade            | **Critical** - If false, skips firmware upgrade entirely |
| `openwrt_packages_upgrade_enabled`        | boolean | `true`  | Upgrade all installed packages via opkg                     | Affects system libraries and applications                |
| `openwrt_packages_upgrade_reboot_enabled` | boolean | `true`  | Reboot device after package upgrades                        | Ensures new packages are loaded                          |
| `openwrt_packages_upgrade_excluded`       | list    | `[]`    | Packages to exclude from upgrade (e.g., `["luci", "dawn"]`) | Prevents breaking custom configurations                  |

### Backup Settings

| Variable                    | Type    | Default    | Description                                         | Impact                                      |
| --------------------------- | ------- | ---------- | --------------------------------------------------- | ------------------------------------------- |
| `openwrt_back_up_enabled`   | boolean | `true`     | Create backup before upgrade                        | **Critical** - Protects configuration data  |
| `openwrt_back_up_local_dir` | string  | _required_ | Local directory for backups (e.g., `"~/Downloads"`) | Where backups are stored on control machine |

### Service Management

| Variable                    | Type | Default | Description                                         | Impact                                    |
| --------------------------- | ---- | ------- | --------------------------------------------------- | ----------------------------------------- |
| `openwrt_disabled_services` | list | `[]`    | Services to disable (e.g., `["odhcpd", "dnsmasq"]`) | Removes DHCP/DNS services from device     |
| `openwrt_enabled_services`  | list | `[]`    | Services to enable (e.g., `["acme", "uhttpd"]`)     | Adds new services like ACME for SSL certs |

### Package Management

| Variable                       | Type | Default | Description                                         | Impact                      |
| ------------------------------ | ---- | ------- | --------------------------------------------------- | --------------------------- |
| `openwrt_installed_packages`   | list | `[]`    | Packages to install (e.g., `["luci", "curl"]`)      | Adds new functionality      |
| `openwrt_uninstalled_packages` | list | `[]`    | Packages to remove (e.g., `["wpad-basic-wolfssl"]`) | Removes unwanted components |

### Advanced Settings

| Variable                        | Type    | Default        | Description                              | Impact                                           |
| ------------------------------- | ------- | -------------- | ---------------------------------------- | ------------------------------------------------ |
| `openwrt_dns_validation_domain` | string  | `"google.com"` | Domain used for DNS resolution test      | Affects network connectivity verification        |
| `openwrt_dns_validation_server` | string  | `"1.1.1.1"`    | Fallback DNS if validation fails         | Used when primary DNS is unreachable             |
| `openwrt_ssh_public_key`        | string  | `~`            | SSH public key to add for access         | Enables passwordless SSH after upgrade           |
| `openwrt_disk_resize_enabled`   | boolean | `false`        | Resize filesystem to max available space | **Warning** - May cause data loss if not careful |

---

## Upgrade Process Flow

### Step 1: Pre-Upgrade Validation

```yaml
# The role performs these checks automatically:
- Validates openwrt_id and openwrt_platform are provided
- Checks device compatibility with selected firmware
- Verifies network connectivity for firmware download
```

**What happens:**

- Role queries OpenWRT Firmware Selector API
- Validates hardware platform matches available firmware
- Ensures device ID is recognized by the project

### Step 2: Backup Creation (if enabled)

```yaml
# If openwrt_back_up_enabled = true:
- Creates full configuration backup
- Stores in specified local directory
- Includes UCI configs, installed packages list
```

**What's backed up:**

- `/etc/config/` - All UCI configurations
- Package installation state
- Network settings, firewall rules, etc.

### Step 3: Firmware Download & Installation

```yaml
# If openwrt_version_upgrade_enabled = true:
- Downloads firmware matching device ID/platform
- Validates checksum integrity
- Installs new firmware via sysupgrade
- Reboots into new system automatically
```

**Behavior notes:**

- Device will be offline during upgrade (~5-15 minutes)
- First boot after upgrade may take 2-3 minutes
- Network settings are preserved if using same device ID

### Step 4: Package Management

```yaml
# If openwrt_packages_upgrade_enabled = true:
- Updates all installed packages via opkg
- Applies package upgrades from enabled_services list
- Removes packages in uninstalled_packages list
- Installs packages in installed_packages list
```

**Important:**

- Excluded packages (`openwrt_packages_upgrade_excluded`) are NOT upgraded
- This prevents breaking custom configurations or third-party apps

### Step 5: Post-Upgrade Actions

```yaml
# If openwrt_packages_upgrade_reboot_enabled = true:
- Reboots device to apply package changes
- Restarts all services automatically
- Validates DNS connectivity
- Applies SSH public key if provided
```

---

## Feature Behavior Matrix

### OpenWRT Version Settings

| `openwrt_version`                    | Behavior                       | Use Case                               |
| ------------------------------------ | ------------------------------ | -------------------------------------- |
| `"latest"`                           | Installs latest stable release | Production environments, general use   |
| `"release_candidate"`                | Includes RC builds as "latest" | Testing new features before stability  |
| `"SNAPSHOT"`                         | Uses development snapshot      | Cutting-edge features, may be unstable |
| Specific version (e.g., `"23.05.4"`) | Installs exact version         | Reproducible deployments, compliance   |

### Service Behavior by Configuration

#### Disabled Services (`openwrt_disabled_services`)

| Service            | When Disabled               | Impact                                    |
| ------------------ | --------------------------- | ----------------------------------------- |
| `odhcpd`           | No DHCP server on device    | Device cannot assign IPs to clients       |
| `dnsmasq`          | No local DNS/DHCP           | External DNS required, no local caching   |
| `kmod-usb-storage` | No USB mass storage support | Cannot mount USB drives via kernel module |

#### Enabled Services (`openwrt_enabled_services`)

| Service                        | When Enabled                        | Impact                                    |
| ------------------------------ | ----------------------------------- | ----------------------------------------- |
| `acme`                         | ACME client for Let's Encrypt certs | Enables automatic SSL certificate renewal |
| `uhttpd`                       | Lightweight HTTP server             | Provides web interface, API access        |
| `prometheus-node-exporter-lua` | System metrics export               | Enables monitoring via Prometheus/Grafana |

### Package Behavior by Configuration

#### Installed Packages (`openwrt_installed_packages`)

Adding packages enables:

- **GUI tools**: `luci`, `htop` - Command-line/system monitoring
- **Networking**: `curl`, `wget` - HTTP utilities
- **Security**: `acme`, `ca-bundle` - SSL/TLS management
- **Monitoring**: `prometheus-node-exporter-lua` - Metrics collection

#### Uninstalled Packages (`openwrt_uninstalled_packages`)

Removing packages disables:

- **Default security**: `wpad-basic-wolfssl` - Removes default Wi-Fi protection
- **Bloatware**: Any pre-installed unwanted components
- **Conflicting software**: Prevents version conflicts with custom builds

### DNS Validation Behavior

| Scenario          | Configuration                                         | Result                                   |
| ----------------- | ----------------------------------------------------- | ---------------------------------------- |
| Primary DNS works | `openwrt_dns_validation_domain` set, server reachable | Normal operation                         |
| Primary DNS fails | Fallback to `openwrt_dns_validation_server`           | Uses configured fallback (e.g., 1.1.1.1) |
| Both fail         | No DNS resolution possible                            | Network connectivity issues until fixed  |

### Backup Behavior

| Setting                           | Result                     | Recovery Option                      |
| --------------------------------- | -------------------------- | ------------------------------------ |
| `openwrt_back_up_enabled = true`  | Full config backup created | Restore from backup if upgrade fails |
| `openwrt_back_up_enabled = false` | No backup created          | Manual recovery required             |

---

## Best Practices

### Before Upgrading Production Devices

1. **Always enable backups**: Set `openwrt_back_up_enabled: true`
2. **Specify backup location**: Configure `openwrt_back_up_local_dir` with accessible path
3. **Test on non-production first**: Validate upgrade process on a spare device
4. **Document current state**: Note all custom configurations before upgrade

### Recommended Configuration for Most Users

```yaml
openwrt_id: "your_device_id"
openwrt_platform: "your_platform"
openwrt_version: "latest"

# Always enable these for safety
openwrt_back_up_enabled: true
openwrt_packages_upgrade_excluded: ["luci", "custom-apps"] # Protect custom configs

# Optional enhancements
openwrt_installed_packages:
  - acme
  - ca-bundle
  - curl
  - htop

openwrt_ssh_public_key: |
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC...
```

### Upgrade Order Recommendations

1. **Single device at a time**: Set `serial: 1` in playbook to upgrade one device per run
2. **Monitor first upgrade**: Watch the initial upgrade for any issues before batch processing
3. **Verify connectivity**: Ensure device reconnects after upgrade before proceeding

### Common Pitfalls to Avoid

| Mistake                                    | Consequence                         | Prevention                                              |
| ------------------------------------------ | ----------------------------------- | ------------------------------------------------------- |
| Missing `openwrt_id` or `openwrt_platform` | Upgrade fails immediately           | Always check OpenWRT Firmware Selector first            |
| Disabling all services accidentally        | Device becomes non-functional       | Review disabled_services list carefully                 |
| Upgrading excluded packages                | Custom configs may break            | Use `openwrt_packages_upgrade_excluded` for custom apps |
| Not enabling backups on production         | No recovery option if upgrade fails | Always enable backups for production devices            |

---

## Troubleshooting

### Upgrade Failed - Check These First

1. **Verify device ID and platform**:

   ```bash
   # Check OpenWRT Firmware Selector: https://firmware-selector.openwrt.org/
   ```

2. **Check backup was created**:

   ```bash
   ls ~/Downloads/OpenWRT/  # Or your configured backup directory
   ```

3. **Review upgrade logs** (if available):
   - Check Ansible task output for specific error messages
   - Look for firmware download failures or checksum mismatches

### After Upgrade Issues

1. **Device won't connect**: May need to reset network settings via console
2. **Custom configs missing**: Restore from backup if `openwrt_back_up_enabled` was true
3. **Services not running**: Check with `/etc/init.d/<service> status`

---

## Summary

| Goal                            | Key Configuration                                               | Important Note                        |
| ------------------------------- | --------------------------------------------------------------- | ------------------------------------- |
| Upgrade firmware only           | `openwrt_version_upgrade_enabled: true`                         | Device will reboot automatically      |
| Keep custom configs intact      | Add to `openwrt_packages_upgrade_excluded`                      | Prevents breaking third-party apps    |
| Protect against upgrade failure | `openwrt_back_up_enabled: true`                                 | Always enable for production devices  |
| Enable new features             | Use `openwrt_installed_packages` and `openwrt_enabled_services` | Review each package/service carefully |

---

_For more information, see the [OpenWRT Firmware Selector](https://firmware-selector.openwrt.org/) to find your device's ID and platform._
