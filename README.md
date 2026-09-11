# Ansible Role: environment

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-environment)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-environment)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-environment)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-environment/dev.yml?branch=dev&event=push&label=dev)](https://github.com/jomrr/ansible-role-environment/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-environment/main.yml?branch=main&event=push&label=main)](https://github.com/jomrr/ansible-role-environment/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for setting up /etc/environment(.d).

## Purpose

Manage named variables in /etc/environment and complete files in
/etc/environment.d. Managed names in /etc/environment have one assignment;
duplicate assignments are consolidated while other names and comments remain.
Declared drop-ins contain exactly their configured entries and are readable
by ordinary users. Applying the same inputs again is idempotent.

## Scope

### Managed

- Declared variable assignments in /etc/environment
- Declared environment.d files and the directory's access permissions

### Not Managed

- Removal of variables or files omitted from the input
- PAM configuration, existing process environments, or service restarts

## Requirements

- systemd with environment.d support is required for drop-ins.
- Login programs must use PAM environment loading to consume /etc/environment.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
```

## Role Variables

### `environment_vars`

Type: `dict`. Required: `false`.

Environment variable names mapped to string values in /etc/environment.
Values must be single-line strings without double quotes, backslashes, hash or
NUL characters for PAM and systemd compatibility.
Unlisted variables and comments are preserved; duplicate assignments for managed
names are consolidated.

Default:

```yaml
environment_vars: {}
```

### `environment_d_files`

Type: `list`. Required: `false`.

Complete environment.d drop-ins, readable by all users.
Unlisted files are preserved.

Default:

```yaml
environment_d_files: []
```

## Managed Files

- `/etc/environment` Named assignments, with root ownership and mode 0644
- `/etc/environment.d/` Root-owned directory with mode 0755; declared files use
  mode 0644

## Check Mode

All resource modules support check mode and report planned changes without
writing files.

## Service Behavior

No services are restarted. PAM reads /etc/environment for new sessions.
systemd reads environment.d when a user manager starts or reloads its
configuration. For an existing user manager, run systemctl --user
daemon-reload in that user's session; subsequently started user services
inherit the updated values. Existing processes retain their environment.

## Security Notes

- These system-wide environment files are public configuration and must not
  contain secrets.
- Changed configuration files receive module-provided backups.

## Operational Notes

- Variable names use ASCII letters, digits and underscores, starting with
  a letter or underscore. Mapping values are strings, including empty strings.
- For compatibility between PAM and systemd, environment_vars values cannot
  contain double quotes, backslashes, hash characters, NUL, CR or LF.
  Use environment_d_files for values requiring quotes, backslashes or newlines.
- Drop-in values retain systemd's dollar-variable expansion. Use $$ for a
  literal dollar sign. PAM does not perform this expansion in /etc/environment.
  Unlike PAM, the systemd generator skips literal empty assignments.
- Drop-ins are loaded in lexical file-name order. Use numerical file-name
  prefixes to control precedence. Names must end in .conf, be unique and
  must not start with a dot.
- No suitable native candidate-file validator is available: the systemd
  environment generator accepts no file argument and can skip malformed
  assignments without failing. Argument validation and domain assertions
  run before any resources are changed.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Configure login and user-service variables

Configure a proxy for new logins and application settings for systemd user services.

```yaml
---
- name: Configure system-wide environment
  hosts: all
  roles:
    - role: jomrr.environment
      environment_vars:
        http_proxy: "http://proxy.example.org:3128"
        no_proxy: "127.0.0.1,localhost"
      environment_d_files:
        - name: 90-application.conf
          entries:
            APPLICATION_LABEL: 'Example "application"'
            APPLICATION_PATH: '/opt/application/bin:$PATH'
```

## References

- [systemd environment.d](https://www.freedesktop.org/software/systemd/man/latest/environment.d.html)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2024 Jonas Mauer.
