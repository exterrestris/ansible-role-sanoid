# Ansible role `exterrestris.sanoid`

An Ansible role to install and configure automated ZFS snapshots and replication using [Sanoid/Syncoid](https://github.com/jimsalterjrs/sanoid) managed by systemd timers, with optional [Healthchecks.io](https://healthchecks.io) support via [runitor](https://github.com/bdd/runitor)

## Requirements

- ZFS
- systemd

In order for Syncoid to replicate to a remote host, you must ensure that SSH access via public key authentication is correctly set up for the relevant users

## Role Variables

### Installation
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `sanoid_install_from` | `"package"` | Install Sanoid from OS package or from GitHub |
| `syncoid_healthchecks_install_runitor` | `false` | Install [runitor](https://github.com/bdd/runitor) for Healthchecks.io support<br>*When set to `true`, will install runitor only if there is at least one `syncoid_syncs` entry with `healthchecks` defined* |

#### Install from source
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `sanoid_source_github_url` | https://github.com/jimsalterjrs/sanoid | GitHub repo to clone |
| `sanoid_source_version` | `latest` | Git branch, tag or commit to checkout. `latest` will select the most recent release |
| `sanoid_source_download_dir` | `/tmp/sanoid` | Directory to clone repo to |
| `sanoid_source_install_dir` | `/usr/local/sbin` | Directory to install binaries to |
| `sanoid_source_remove_package` | `yes` | Remove the OS package if installed |

#### Runitor/Healthchecks
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `syncoid_healthchecks_runitor_github_url` | `https://github.com/bdd/runitor` | GitHub repo to download latest release from |
| `syncoid_healthchecks_runitor_install_dir` | `/usr/local/sbin` | Directory to install runitor binary to |


### Sanoid Configuration
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `sanoid_templates` | Example templates from [sanoid.conf](https://github.com/jimsalterjrs/sanoid/blob/master/sanoid.conf) | List of policy templates |
| `sanoid_datasets` | `[]` | List of datasets to snapshot |

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

#### systemd Settings
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `sanoid_service_pre_start` | `[]` | Add ExecStartPre commands to the Sanoid systemd service file |
| `sanoid_service_post_start` | `[]` | Add ExecStartPost commands to the Sanoid systemd service file |
| `sanoid_prune_service_pre_start` | `[]` | Add ExecStartPre commands to the Sanoid Prune systemd service file |
| `sanoid_prune_service_post_start` | `[]` | Add ExecStartPost commands to the Sanoid Prune systemd service file |


### Syncoid Configuration
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `syncoid_syncs` | `[]` | List of datasets to replicate |
| `syncoid_send_options` | `""` | Default additional options to pass to the zfs send command via `syncoid --sendoptions` |
| `syncoid_recv_options` | `""` | Default additional options to pass to the zfs recieve command via `syncoid --recvoptions` |
| `syncoid_use_ssh_key` | `yes` | Use an SSH key to login to remote hosts |
| `syncoid_generate_ssh_key` | `yes` | Generate an SSH key for Syncoid to use |
| `syncoid_generated_ssh_key` | `id_syncoid` | Name of generated SSH key |
| `syncoid_ssh_key` | `/root/.ssh/{`*`syncoid_generated_ssh_key`*` \| id_rsa }` | Path to SSH key for Syncoid to use |
| `syncoid_ssh_key_install_remote` | `yes` | Install specified SSH key on remote hosts. Requires remote hosts to be defined in inventory |
| `syncoid_update_known_hosts` | `yes` | Update known_hosts with src/destination host public keys. |

#### `syncoid_syncs[]`
A separate systemd service will be generated for each sync listed, grouped into a single systemd target triggered by a systemd timer. The generated services are configured to run one at a time, in the order they are listed

| Variable | Default | Comments |
| :--- | :--- | :--- |
| `name` | Auto-generated | Name to use for the generated systemd service.<br>*Specifying a human-readable name is highly recommended* |
| `src` | *Required* | Source ZFS dataset |
| `src_host` | `""` | Source host |
| `src_user` | `"root"` | Source user. Ignored if `src_host` empty |
| `dest` | *Required* | Destination ZFS dataset |
| `dest_host` | `""` | Destination host |
| `dest_user` | `"root"` | Destination user. Ignored if `dest_host` empty |
| `recursive` | `"no"` | Copy child datasets |
| `force_delete` | `"no"` | Remove destination datasets recursively |
| `send_options` | `""` | Additional options to pass to the zfs send command via `syncoid --sendoptions` |
| `recv_options` | `""` | Additional options to pass to the zfs recieve command via `syncoid --recvoptions` |

#### systemd Settings
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `syncoid_service_name` | `"syncoid"` | Name to use for the Syncoid systemd target, and the prefix to use for the individual systemd sync services |
| `syncoid_timer_frequency` | `"daily"` | systemd service frequency for Syncoid |
| `syncoid_syncs[].systemd.pre_start` | `[]` | Add ExecStartPre commands to the systemd service file for the specific sync |
| `syncoid_syncs[].systemd.post_start` | `[]` | Add ExecStartPost commands to the systemd service file for the specific sync |


### Healthchecks.io
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `syncoid_healthchecks_api_url` | `"https://hc-ping.com"` | Default Healthchecks.io API URL to use |
| `syncoid_healthchecks_ping_key` | `""` | Default project ping key to use |
| `syncoid_syncs[].healthchecks.api_url` | `{{ syncoid_healthchecks_api_url }}` | API URL to use for the specific sync |
| `syncoid_syncs[].healthchecks.ping_key` | `{{ syncoid_healthchecks_ping_key }}` | Project ping key to use for the specific sync |
| `syncoid_syncs[].healthchecks.slug` | Required | Check slug to use for the specific sync |

## Example

```Yaml
syncoid_healthchecks_install_runitor: true
syncoid_healthchecks_ping_key: **project-ping-key**

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
  - name: main-backup
    src: zpoolname/parent
    dest: zpoolname/parent-backup
    dest_host: remote
    recursive: yes
    # Healthchecks.io ping via runitor
    healthchecks:
      slug: main-backup
  - src: zpoolname/dataset
    dest: zpoolname/dataset-backup
    # Manual Healthchecks.io ping via curl
    systemd:
      pre_start:
        - "/usr/bin/curl https://hc-ping.com/**uuid**/start"
      post_start:
        - "/usr/bin/curl https://hc-ping.com/**uuid**"
```
