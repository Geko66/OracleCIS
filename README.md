# Oracle Linux 10 CIS Hardening with Ansible

Production-grade Ansible project for Oracle Linux 10 CIS hardening with modular roles for:

- `common`
- `users`
- `ssh`
- `firewall`
- `selinux`
- `audit`
- `oscap`

## What this project does

- Applies CIS-oriented Level 1 controls by default.
- Enables extra Level 2 controls when `cis_enable_level2: true`.
- Uses `authselect` safely for PAM-driven controls on Oracle Linux 10.
- Configures OpenSCAP with `scap-security-guide`.
- Schedules recurring scans with systemd timers by default, or cron when requested.
- Writes HTML and XML scan artifacts to `/var/log/oscap`.
- Generates remediation artifacts in Bash and Ansible formats after scans.
- Supports immediate remediation during scan runs when explicitly enabled.

## Important OpenSCAP note

Oracle documents `openscap`, `openscap-utils`, and `scap-security-guide` for Oracle Linux 10. However, Oracle-published OL10 guidance does not clearly guarantee that Oracle-specific CIS profiles are shipped in every `ssg-ol10-ds.xml` build. Because of that, this project:

- hardens the host directly with Ansible to a CIS-oriented baseline
- auto-detects CIS OpenSCAP profiles when they exist
- fails clearly if no CIS profile is available and `oscap_fail_when_cis_profile_missing` remains `true`
- supports external datastream and profile overrides through variables

## Install collections

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

## Run the hardening playbook

```bash
ansible-playbook playbooks/hardening.yml
```

## Run an immediate compliance scan

```bash
ansible-playbook playbooks/oscap-scan.yml
```

## Run a scan with remediation enabled

```bash
ansible-playbook playbooks/oscap-remediate.yml
```

## Key variables

Set these in [`group_vars/all.yml`](./group_vars/all.yml):

- `cis_enable_level2`
- `cis_install_security_updates_automatically`
- `cis_local_users`
- `cis_firewalld_services`
- `cis_firewalld_ports`
- `oscap_profile_id`
- `oscap_datastream_path_override`
- `oscap_tailoring_path`
- `oscap_apply_remediation_during_scan`
