# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

~

## [1.3.0] - 2023-09-04

### Added

- Option to resize the filesystem to max available (e.g., SD-card)
- Option to disable version upgrade of device
- Managed installation and execution of the [`gekmihesg.openwrt`](https://galaxy.ansible.com/gekmihesg/openwrt) role
- [`ansible-lint`](https://github.com/ansible/ansible-lint) configuration
- Conditionals for task-file inclusions

### Changed

- Optimized `opkg` cache updates

### Fixed

- Linting issues
- Post-upgrade version comparison issue

## [1.2.0] - 2023-08-12

### Added

- Option to set `openwrt_version` to `"release_candidate"` to consider release candidate builds as 'latest'
- Output to show the configuration of the role execution (useful when lists are assembled)
- Advanced playbook example to manage multiple devices in one playbook
- Allowed inline HTML elements to [`markdown`](https://en.wikipedia.org/wiki/Markdown) linting configuration

### Changed

- Release candidates version no longer considered for `openwrt_version: "latest"`

### Fixed

- Interchange `/vars` with `/defaults` to conform to standards

## [1.1.0] - 2023-04-07

### Added

- Files to setup development environment using [`direnv`](https://direnv.net)
- Linting configuration for [`markdown`](https://en.wikipedia.org/wiki/Markdown)
- Handler to reboot device
- Upgrade packages functionality

### Changed

- Configuration settings for [`yamllint`](https://yamllint.readthedocs.io/en/stable/)
- Update `opkg` cache only once and loop over packages instead of joining them

### Fixed

- Linting issues
- [Check mode](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_checkmode.html#using-check-mode)
- Variable comments

## [1.0.0] - 2022-10-24

### Added

- Check to get the latest stable OpenWRT version
- Output step to show which OpenWRT version is about to be installed

### Changed

- Default value for `openwrt_version` to `latest`
- Clarification throughout with output references to `version` and `version code`

### Fixed

- Linting issues

## [0.0.3] - 2022-10-23

### Changed

- Clarified currently installed version for OpenWRT version output

### Fixed

- Variable name for `sysupgrade` file path in `defaults`
- GitHub versioning tags to actually conform to [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html)
- Role metadata schema issues

## [0.0.2] - 2022-04-04

### Added

- Back-up functionality
- `undef` keyword to required variables

## [0.0.1] - 2022-04-04

### Added

- Initial release

[Unreleased]: https://github.com/jorneilander/ansible-role-openwrt/compare/1.3.0...HEAD
[1.3.0]: https://github.com/jorneilander/ansible-role-openwrt/compare/1.2.0...1.3.0
[1.2.0]: https://github.com/jorneilander/ansible-role-openwrt/compare/1.1.0...1.2.0
[1.1.0]: https://github.com/jorneilander/ansible-role-openwrt/compare/1.0.0...1.1.0
[1.0.0]: https://github.com/jorneilander/ansible-role-openwrt/compare/0.0.3...1.0.0
[0.0.3]: https://github.com/jorneilander/ansible-role-openwrt/compare/0.0.2...0.0.3
[0.0.2]: https://github.com/jorneilander/ansible-role-openwrt/compare/0.0.1...0.0.2
[0.0.1]: https://github.com/jorneilander/ansible-role-openwrt/releases/tag/0.0.1
