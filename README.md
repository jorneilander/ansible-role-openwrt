# OpenWRT

A small [Ansible](https://www.ansible.com/) role to manage your [OpenWRT](https://openwrt.org/) devices.
It will check for updates using the [OpenWRT Firmware Selector](https://firmware-selector.openwrt.org/) for your device, and apply the latest version if yours doesn't have it installed.

You can also manage any packages and services you want to have installed or removed, as well as add an authorized SSH key for easy access.

## Requirements

You will need to run this role from a playbook, and have `gather_facts` enabled.
Otherwise [`gekmihesg.openwrt`](https://galaxy.ansible.com/gekmihesg/openwrt) will not be able to run Ansible tasks on OpenWRT.

## Role Variables

| Variable                       | type      | default  | example                                              | description                                                                                                                        |
| ------------------------------ | --------- | -------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `openwrt_version`              | `string`  | `latest` | `"SNAPSHOT"`                                         | OpenWRT version to be installed. Use `SNAPSHOT` if there are no versioned builds. Defaults to latest version of OpenWRT available. |
| `openwrt_platform`             | `string`  | `~`      | `"realtek/rtl838x"`                                  | OpenWRT platform for your device. Check [OpenWRT Firmware Selector](https://firmware-selector.openwrt.org/) for your device.       |
| `openwrt_id`                   | `string`  | `~`      | `"netgear_gs308t-v1"`                                | OpenWRT id for your device. Check [OpenWRT Firmware Selector](https://firmware-selector.openwrt.org/) for your device.             |
| `openwrt_disabled_services`    | `list`    | `[]`     | `["odhcpd", "dnsmasq"]`                              | List of services to be disabled on your device.                                                                                    |
| `openwrt_enabled_services`     | `list`    | `[]`     | `["acme", "uhttpd", "prometheus-node-exporter-lua"]` | List of services to be enabled on your device.                                                                                     |
| `openwrt_installed_packages`   | `list`    | `[]`     | `["acme", luci", "luci-app-acme", "wpad-wolfssl"]`   | List of packages to be installed on your device.                                                                                   |
| `openwrt_uninstalled_packages` | `list`    | `[]`     | `["wpad-basic-wolfssl"]`                             | List of packages to uninstalled on your device.                                                                                    |
| `openwrt_ssh_public_key`       | `string`  | `~`      | `"ssh-rsa AAAAB3N...f6f+K8="`                        | SSH public-key to be added to the list of Dropbear authored keys.                                                                  |
| `openwrt_back_up_enabled`      | `boolean` | `true`   | `true`                                               | Create back-up of the OpenWRT configuration before updating.                                                                       |
| `openwrt_back_up_local_dir`    | `string`  | `~`      | `"~/Downloads"`                                      | Directory on your local machine to store back-ups.                                                                                 |

## Dependencies

This role relies on the [`gekmihesg.openwrt`](https://galaxy.ansible.com/gekmihesg/openwrt) to allow managing OpenWRT without having Python installed on the device.

My thanks to [gekmihesg](https://github.com/gekmihesg)!

## Example Playbook

```YAML
- hosts: openwrt_switch:openwrt_accesspoint
  gather_facts: true
  serial: 1
  tasks:
    - name: "[OpenWRT] - Import role"
      ansible.builtin.import_role:
        name: openwrt
      vars:
        openwrt_version: "latest"
        openwrt_platform: "ramips/mt7621"
        openwrt_id: "xiaomi_mi-router-ac2100"
        openwrt_disabled_services:
          - "odhcpd"
          - "dnsmasq"
        openwrt_enabled_services:
          - "acme"
          - "uhttpd"
          - "prometheus-node-exporter-lua"
        openwrt_installed_packages:
          - acme
          - ca-bundle
          - coreutils-base64
          - coreutils-sha1sum
          - curl
          - htop
          - luaposix
          - luci
          - luci-app-acme
          - luci-app-uhttpd
          - luci-mod-rpc
          - prometheus-node-exporter-lua
          - prometheus-node-exporter-lua-openwrt
          - uhttpd
          - uhttpd-mod-ubus
        openwrt_uninstalled_packages:
          - "wpad-basic-wolfssl"
        openwrt_ssh_public_key: ~
```

## License

[GPL-3.0-or-later](https://spdx.org/licenses/GPL-3.0-or-later.html)

## Author Information

[Jorn Eilander](https://github.com/jorneilander)
