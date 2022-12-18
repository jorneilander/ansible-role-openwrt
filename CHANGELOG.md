# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Files to setup development environment using [`direnv`](https://direnv.net)
- Linting configuration for [`markdown`](https://en.wikipedia.org/wiki/Markdown)

### Changed

- Configuration settings for [`yamllint`](https://yamllint.readthedocs.io/en/stable/)

### Fixed

- Linting issues
- [Check mode](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_checkmode.html#using-check-mode)

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

[Unreleased]: https://github.com/jorneilander/ansible-role-openwrt/compare/1.0.0...HEAD
[1.0.0]: https://github.com/jorneilander/ansible-role-openwrt/compare/0.0.3...1.0.0
[0.0.3]: https://github.com/jorneilander/ansible-role-openwrt/compare/0.0.2...0.0.3
[0.0.2]: https://github.com/jorneilander/ansible-role-openwrt/compare/0.0.1...0.0.2
[0.0.1]: https://github.com/jorneilander/ansible-role-openwrt/releases/tag/0.0.1
