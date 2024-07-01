# Ansible role `exterrestris.sanoid`

An Ansible role to install and configure automated ZFS snapshots and replication using [Sanoid/Syncoid](https://github.com/jimsalterjrs/sanoid)

## Requirements

- Sanoid package available in distribution
- systemd

In order for Syncoid to replicate to a remote host, you must ensure that SSH access via public key authentication is correctly set up for the relevant users

## Role Variables

### Installation
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `sanoid_install_from` | `"package"` | Install Sanoid from OS package or from GitHub |

#### Install from source
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `sanoid_source_github_url` | https://github.com/jimsalterjrs/sanoid | GitHub repo to clone |
| `sanoid_source_version` | `latest` | Git branch, tag or commit to checkout. `latest` will select the most recent release |
| `sanoid_source_download_dir` | `/tmp/sanoid` | Directory to clone repo to |
| `sanoid_source_install_dir` | `/usr/local/sbin` | Directory to install binaries to |
| `sanoid_source_remove_package` | `yes` | Remove the OS package if installed |

### Configuration
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `sanoid_datasets` | `[]` | List of datasets to snapshot |
| `sanoid_templates` | Example templates from [sanoid.conf](https://github.com/jimsalterjrs/sanoid/blob/master/sanoid.conf) | List of policy templates |
| `syncoid_syncs` | `[]` | List of datasets to replicate |

#### `sanoid_templates[]`
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `name` | *Required* | Template name |
| *`setting`* | `""` | Policy setting |

All settings supported by Sanoid in templates are supported - see [sanoid.conf](https://github.com/jimsalterjrs/sanoid/blob/master/sanoid.conf) and [sanoid.defaults.conf](https://github.com/jimsalterjrs/sanoid/blob/master/sanoid.defaults.conf) for details
Similarly, most [Syncoid flags](https://github.com/jimsalterjrs/sanoid/wiki/Syncoid#options) are configurable via `syncoid_syncs`.

#### `sanoid_datasets[]`
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `name` | *Required* | ZFS dataset to snapshot |
| `templates` | *Required* | Sanoid template(s) to use for policy |
| `recursive` | `"no"` | Include child datasets with this dataset |
| `process_children_only` | `"no"` | Do not include this dataset |
| `overrides` | `[]` | List of template settings to override |

#### `syncoid_syncs[]`
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `src` | *Required* | Source ZFS dataset |
| `src_host` | `""` | Source host |
| `src_user` | `"root"` | Source user. Ignored if `src_host` empty |
| `dest` | *Required* | Destination ZFS dataset |
| `dest_host` | `""` | Destination host |
| `dest_user` | `"root"` | Destination user. Ignored if `dest_host` empty |
| `recursive` | `"no"` | Copy child datasets |
| `force_delete` | `"no"` | Remove destination datasets recursively |

### Syncoid systemd Settings
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `syncoid_service_name` | `"syncoid"` | systemd service name for Syncoid |
| `syncoid_timer_frequency` | `"daily"` | systemd service frequency for Syncoid |
| `syncoid_use_ssh_key` | `yes` | Use an SSH key to login to remote hosts |
| `syncoid_generate_ssh_key` | `yes` | Generate an SSH key for Syncoid to use |
| `syncoid_generated_ssh_key` | `id_syncoid` | Name of generated SSH key |
| `syncoid_ssh_key` | `/root/.ssh/{`*`syncoid_generated_ssh_key`*`\|id_rsa}` | Path to SSH key for Syncoid to use |
| `syncoid_ssh_key_install_remote` | `yes` | Install specified SSH key on remote hosts. Requires remote hosts to be defined in inventory |
| `syncoid_update_known_hosts` | `yes` | Update known_hosts with src/destination host public keys. |

## Example

```Yaml
sanoid_templates:
  - name: production
    frequently: 0
    hourly: 36
    daily: 30
    monthly: 3
    yearly: 0
    autosnap: 'yes'
    auto prune: 'yes'
  - name: backup
    frequently: 0
    hourly: 30
    daily: 90
    monthly: 12
    yearly: 0
    autoprune: 'yes'
    autosnap: 'no'
  - name: ignore
    autoprune: 'no'
    autosnap: 'no'
    monitor: 'no'

sanoid_datasets:
  - name: zpoolname/dataset
    templates:
      - production
      - demo
    overrides:
      hourly: 12
      monthly: 1
  - name: zpoolname/parent
    templates: production
    recursive: 'yes'
    process_children_only: 'yes'
  - name: zpoolname/parent/child
    templates: demo
    overrides:
      hourly: 4

syncoid_syncs:
  - src: zpoolname/parent
    dest: zpoolname/parent-backup
    dest_host: remote
    recursive: yes
  - src: zpoolname/dataset
    dest: zpoolname/dataset-backup
```
