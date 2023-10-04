# OpenWRT

A small [Ansible](https://www.ansible.com/) role to manage your [OpenWRT](https://openwrt.org/) devices.
It will check for updates using the [OpenWRT Firmware Selector](https://firmware-selector.openwrt.org/) for your device,
and apply the latest version if yours doesn't have it installed.

You can also manage any packages and services you want to have installed or removed,
as well as add an authorized SSH key for easy access.

## Requirements

You will need to run this role from a playbook, and have `gather_facts` enabled.
Otherwise [`gekmihesg.openwrt`](https://galaxy.ansible.com/gekmihesg/openwrt) will not be able to run Ansible tasks on OpenWRT.

## Role Variables

| Variable                                  | type      | default  | example                                              | description                                                                                                                  |
| ----------------------------------------- | --------- | -------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `openwrt_version`                         | `string`  | `latest` | `"SNAPSHOT"`                                         | OpenWRT version to be installed. [More information](#openwrt_version)                                                        |
| `openwrt_platform`                        | `string`  | `~`      | `"realtek/rtl838x"`                                  | OpenWRT platform for your device. Check [OpenWRT Firmware Selector](https://firmware-selector.openwrt.org/) for your device. |
| `openwrt_id`                              | `string`  | `~`      | `"netgear_gs308t-v1"`                                | OpenWRT id for your device. Check [OpenWRT Firmware Selector](https://firmware-selector.openwrt.org/) for your device.       |
| `openwrt_version_upgrade_enabled`         | `boolean` | `true`   | `true`                                               | Enable option to upgrade OpenWRT version of the device                                                                       |
| `openwrt_disabled_services`               | `list`    | `[]`     | `["odhcpd", "dnsmasq"]`                              | List of services to be disabled on your device.                                                                              |
| `openwrt_enabled_services`                | `list`    | `[]`     | `["acme", "uhttpd", "prometheus-node-exporter-lua"]` | List of services to be enabled on your device.                                                                               |
| `openwrt_packages_upgrade_enabled`        | `boolean` | `true`   | `true`                                               | Enable option to upgrade all installed packages if possible                                                                  |
| `openwrt_packages_upgrade_reboot_enabled` | `boolean` | `true`   | `true`                                               | Reboot device after packages have been upgraded                                                                              |
| `openwrt_packages_upgrade_excluded`       | `list`    | `[]`     | `["luci", "dawn"]`                                   | Exclude certain packages from being upgraded                                                                                 |
| `openwrt_installed_packages`              | `list`    | `[]`     | `["acme", luci", "luci-app-acme", "wpad-wolfssl"]`   | List of packages to be installed on your device.                                                                             |
| `openwrt_uninstalled_packages`            | `list`    | `[]`     | `["wpad-basic-wolfssl"]`                             | List of packages to uninstalled on your device.                                                                              |
| `openwrt_ssh_public_key`                  | `string`  | `~`      | `"ssh-rsa AAAAB3N...f6f+K8="`                        | SSH public-key to be added to the list of Dropbear authored keys.                                                            |
| `openwrt_back_up_enabled`                 | `boolean` | `true`   | `true`                                               | Create back-up of the OpenWRT configuration before updating.                                                                 |
| `openwrt_back_up_local_dir`               | `string`  | `~`      | `"~/Downloads"`                                      | Directory on your local machine to store back-ups.                                                                           |

### `openwrt_version`

Set to a specific version of OpenWRT to be installed on your device (e.g., `"22.03.5"`).
Defaults to latest stable version of OpenWRT available (i.e., `"latest"`).
Use `"release_candidate"` to consider release candidate builds as latest.

Use `"SNAPSHOT"` if there are no versioned builds for your device.

## Dependencies

This role relies on the [`gekmihesg.openwrt`](https://galaxy.ansible.com/gekmihesg/openwrt) to allow managing OpenWRT
without having Python installed on the device.

My thanks to [gekmihesg](https://github.com/gekmihesg)!

## Example Playbook (simple)

```YAML
- hosts: openwrt_accesspoint
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

## Example Playbook (advanced)

The following (extensive) example shows how to use this role to manage an entire fleet of OpenWRT devices.

<details>
  <summary>Click to show the example</summary>

By leveraging the [`lookup(ansible.builtin.varnames)`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/varnames_lookup.html) plugin we can assemble the lists we need as input for the role execution.
By concatenating lists ending with a certain string (e.g., `*openwrt_installed_packages`) we can easily manage what gets installed in which (group of) device(s).
We only need to be mindful of not re-using variable names.
In this example all variables for a group have the prefix `group_<group_name>_` to mitigate that issue.

- Every individual device gets their device configuration set in `inventory/hosts.yml`.
- Defaults for _all_ devices (e.g., installing `luci`) get set using the `openwrt` group variables in `inventory/group_vars/openwrt.yaml`.
- Defaults for device groups (e.g., access points) get set using the their respective group variables (e.g., `inventory/group_vars/openwrt_access_point.yaml`)

<details>
  <summary>Click to show all the example files</summary>

```YAML
# inventory/hosts.yaml
---
all:
  hosts:
    switch1:
      ansible_host: 10.0.0.2
      openwrt:
        id: netgear_gs308t-v1
        platform: "realtek/rtl838x"
        version: latest
    switch2:
      ansible_host: 10.0.0.3
      openwrt:
        id: netgear_gs308t-v1
        platform: "realtek/rtl838x"
        version: latest
    ap1:
      ansible_host: 10.0.0.4
      openwrt:
        id: xiaomi_ax3600
        platform: "ipq807x/generic"
        version: release_candidate
    ap2:
      ansible_host: 10.0.0.5
      openwrt:
        id: xiaomi_mi-router-ac2100
        platform: "ramips/mt7621"
        version: latest
  children:
    openwrt:
      children:
        openwrt_switch:
        openwrt_accesspoint:
    openwrt_switch:
      hosts:
        switch1:
        switch2:
    openwrt_accesspoint:
      hosts:
        ap1:
        ap2:
```

```YAML
# inventory/group_vars/openwrt.yaml
---
# Settings applicable to all devices
ansible_user: root
ansible_scp_extra_args: "-O"

group_openwrt_openwrt_enabled_services:
  - uhttpd

group_openwrt_openwrt_disabled_services: []

group_openwrt_openwrt_installed_packages:
  - acme
  - luci
  - luci-app-acme
  - luci-app-nlbwmon
  - luci-app-uhttpd
  - uhttpd
  - uhttpd-mod-ubus

group_openwrt_openwrt_uninstalled_packages: []
```

```YAML
# inventory/group_vars/openwrt_accesspoint.yaml
---
# Settings applicable to access points only
group_access_point_openwrt_installed_packages:
  - dawn
  - luci-app-dawn
  - wpad-wolfssl

group_access_point_openwrt_uninstalled_packages:
  - wpad-basic-mbedtls
  - wpad-basic-wolfssl

group_access_point_openwrt_enabled_services: []

group_access_point_openwrt_disabled_services:
  - dnsmasq
  - odhcpd
```

```YAML
# inventory/group_vars/openwrt_switch.yaml
---
# Settings applicable to switches only
group_switch_openwrt_installed_packages: []
group_switch_openwrt_uninstalled_packages: []

group_switch_openwrt_enabled_services: []
group_switch_openwrt_disabled_services:
  - dnsmasq
  - odhcpd
```

The final assembly takes place in the playbook itself.

```YAML
# playbook.yaml
- name: "Patch OpenWRT devices"
  hosts: openwrt
  gather_facts: true
  serial: 1

  tasks:
    - name: "[OpenWRT] - Gather packages to be uninstalled"
      ansible.builtin.set_fact:
        openwrt_uninstalled_packages: >
          {{ openwrt_uninstalled_packages | default([]) + vars[item] }}
      loop: "{{ lookup('varnames', '.*openwrt_uninstalled_packages').split(',') }}"

    - name: "[OpenWRT] - Gather services to be enabled"
      ansible.builtin.set_fact:
        openwrt_enabled_services: >
          {{ openwrt_enabled_services | default([]) + vars[item] }}
      loop: "{{ lookup('varnames', '.*openwrt_enabled_services').split(',') }}"

    - name: "[OpenWRT] - Gather services to be disabled"
      ansible.builtin.set_fact:
        openwrt_disabled_services: >
          {{ openwrt_disabled_services | default([]) + vars[item] }}
      loop: "{{ lookup('varnames', '.*openwrt_disabled_services').split(',') }}"

    - name: "[OpenWRT] - Gather packages to be installed"
      ansible.builtin.set_fact:
        openwrt_installed_packages: >
          {{ openwrt_installed_packages | default([]) + vars[item] }}
      loop: "{{ lookup('varnames', '.*openwrt_installed_packages').split(',') }}"

    - name: "[OpenWRT] - Import role"
      ansible.builtin.import_role:
        name: ansible-role-openwrt
      vars:
        openwrt_version: "{{ openwrt.version | default('latest') }}"
        openwrt_platform: "{{ openwrt.platform }}"
        openwrt_id: "{{ openwrt.id }}"
        openwrt_disabled_services: "{{ openwrt_disabled_services | default([]) }}"
        openwrt_enabled_services: "{{ openwrt_enabled_services | default([]) }}"
        openwrt_installed_packages: "{{ openwrt_installed_packages | default([]) }}"
        openwrt_uninstalled_packages: "{{ openwrt_uninstalled_packages | default([]) }}"
        openwrt_back_up_local_dir: "~/Downloads/OpenWRT"
```

</details>
</details>

## Contributing

### Required software

- `Python 3`
- [`direnv`](https://direnv.net)

### Setting up your development environment

Running `direnv allow` should setup your development environment for this role.

## License

[GPL-3.0-or-later](https://spdx.org/licenses/GPL-3.0-or-later.html)

## Author Information

[Jorn Eilander](https://github.com/jorneilander)
