# 📚 Homelab Proxmox Ansible

Ansible playbooks for post-install, hardening and maintenance of a Proxmox VE homelab.

Tailored for single-node, non-HA, non-subscription and non-enterprise hardware setups.
The playbooks streamline initial configuration, enhance security, and ensure smooth
upgrades and maintenance.

**Table of contents**

- [✅ Requirements](#-requirements)
- [📦 Variables](#-variables)
- [🌀 Handlers](#-handlers)
- [🏷️ Tags](#-tags)
- [🧪 Usage](#-usage)

## ✅ Requirements

- Proxmox VE `9.x` host reachable via SSH.
- Control node with Ansible `2.20+` and access to the SSH public key declared in
  `ssh_pub_key_path`.
- `sudo` privileges on the target host.

## 📦 Variables

Defaults live in each role under `defaults/main.yaml`; override from inventory,
group vars, or host vars when tinkering.

| Variable                         | Purpose                                                      |
| -------------------------------- | ------------------------------------------------------------ |
| `release_codename`               | Debian/Proxmox suite used when templating repo definitions.  |
| `apt_architecture`               | CPU architecture list for repository entries.                |
| `gpg_keyring_path`               | Keyring file shipped with Proxmox for signed repos.          |
| `apt_repositories_to_remove`     | Deb822 entries scrubbed to drop subscription feeds.          |
| `apt_repositories_to_add`        | Deb822 entries injected for community feeds.                 |
| `apt_packages_to_install`        | Baseline lab tooling installed during fresh installs.        |
| `ha_cluster_services_to_disable` | HA/cluster daemons stopped to avoid useless churn on solos.  |
| `trim_timer_unit`                | Systemd timer unit that keeps SSD trim enabled.              |
| `journald_conf_path`             | Target `journald.conf` file for RAM-only logging tweaks.     |
| `journald_storage_regex`         | Regex used to locate/replace the journald storage directive. |
| `journald_storage_value`         | Desired journald storage setting (`Storage=volatile`).       |
| `ssh_user`                       | Default account whose key gets authorized.                   |
| `ssh_pub_key_path`               | Public key on the control node pushed to the host.           |
| `ssh_authorized_accounts`        | List of user/key pairs enforced via `authorized_key`.        |
| `sshd_config_path`               | Config file hardened with opinionated directives.            |
| `sshd_directives`                | Regex/line pairs applied to lock down SSH.                   |

## 🌀 Handlers

- `Restart systemd-journald` – triggered when journald storage is changed.
- `Restart sshd` – async restart to apply hardened settings without hanging the run.

## 🏷️ Tags

| Tag                | Scope                                           | Typical run                        |
| ------------------ | ----------------------------------------------- | ---------------------------------- |
| `repos`            | Repository CRUD plus cache refresh              | Convert feeds before upgrades.     |
| `lifecycle`        | Dist-upgrade and tooling packages               | Routine patch windows.             |
| `ssd_health`       | HA disablement, trim timer, journald-in-RAM     | SSD wear mitigation.               |
| `access`           | Authorized keys + SSH hardening                 | Re-assert access policies.         |
| `non_subscription` | Repo removal/addition + cache refresh           | Ditch enterprise feeds.            |
| `fresh_install`    | Lifecycle bootstrapping on day-zero nodes       | Immediately after imaging.         |
| `upgrade`          | Lifecycle dist-upgrade without bootstrap extras | Monthly patches.                   |
| `single_node`      | HA and cluster service disablement              | Solo Proxmox nodes only.           |
| `ssh_hardening`    | Key enforcement + `sshd_config` directives      | Key rotations or policy refreshes. |

## 🧪 Usage

No need to provide inventory as it's picked up from `ansible.cfg` by default.

```bash
# Full configuration
ansible-playbook playbooks/main.yaml

# Run an upgrade lifecycle only
ansible-playbook playbooks/upgrade.yaml
```

Inventory Snippet.

```ini
[proxmox]
bm-01 ansible_host=<your-host-ip>

[proxmox:vars]
ansible_user=root
ansible_python_interpreter=/usr/bin/python3
```

## 📜 License

Released under the MIT License. See `LICENSE` for the full text.

## 👤 Author

[Jaime de Diego](mailto:art-breach-capitol@duck.com)
