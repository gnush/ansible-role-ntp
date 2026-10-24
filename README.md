# NTP (chrony) Ansible Role

Installs and configures [chrony](https://chrony-project.org/) as the NTP daemon.

## Requirements

- Ubuntu >= 22.04
- Ansible >= 2.11
- Root privileges (`become: true`)

## Role Variables

Defined in `defaults/main.yml`:

| Variable                 | Default                   | Description                                          |
|--------------------------|---------------------------|------------------------------------------------------|
| `ntp_service_name`       | `chronyd`                 | Name of the chrony service                           |
| `ntp_config_file`        | `/etc/chrony/chrony.conf` | Path of the main chrony config file                  |
| `ntp_sources_dir`        | `/etc/chrony/sources.d`   | Directory for `*.sources` files                      |
| `ntp_conf_d_dir`         | `/etc/chrony/conf.d`      | Directory for `*.conf` fragments                     |
| `ntp_allow_client_cidrs` | `[]`                      | Client ranges allowed to use this host as NTP server |
| `ntp_sources`            | `pool.ntp.org`, `iburst`  | Upstream time sources                                |

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - ntp
```

## Chrony Configuration

`templates/chrony.conf.j2` is based on the default Ubuntu config.

### Time Sources

Each entry of `ntp_sources` becomes one line `<type> <host> <options...>`.
The default is the public [NTP Pool](https://www.ntppool.org/):

```yaml
ntp_sources:
  - type: pool            # pool | server
    host: pool.ntp.org
    options: [iburst]     # optional, see `man chrony.conf`
  - type: server
    host: time.cloudflare.com
    options: [iburst, nts]
```

Options are passed through unchecked. When the list is empty, `ntp.sources` is
removed and chrony has no configured upstream.

### Client IP-Ranges

Set `ntp_allow_client_cidrs` to let clients from these ranges use the host as an NTP server.
When the list is empty (default), `allow.conf` is removed and the server function is off.

```yaml
ntp_allow_client_cidrs:
  - 10.0.0.0/8      # RFC 1918
  - 172.16.0.0/12   # RFC 1918
  - 192.168.0.0/16  # RFC 1918
  - 169.254.0.0/16  # IPv4 link-local (RFC 3927)
```
