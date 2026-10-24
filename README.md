# NTP (chrony) Ansible Role

Installs and configures [chrony](https://chrony-project.org/) as the NTP daemon.
The host syncs its clock from public NTP pool servers and also acts as an NTP
server for clients on private and link-local networks.

## Requirements

- Ubuntu >= 22.04
- Ansible >= 2.10
- Root privileges (`become: true`)

## Role Variables

Defined in `defaults/main.yml`:

| Variable           | Default                    | Description                         |
|--------------------|----------------------------|-------------------------------------|
| `ntp_service_name` | `chronyd`                  | Name of the chrony service          |
| `ntp_config_file`  | `/etc/chrony/chrony.conf`  | Path of the main chrony config file |
| `ntp_sources_dir`  | `/etc/chrony/sources.d`    | Directory for `*.sources` files     |
| `ntp_conf_d_dir`   | `/etc/chrony/conf.d`       | Directory for `*.conf` fragments    |

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - ntp
```

## Chrony Configuration

`templates/chrony.conf.j2` is based on the default Ubuntu config.

Every `*.conf` file in `files/` is copied to `ntp_conf_d_dir`. Every
`*.sources` file is copied to `ntp_sources_dir`. To add config, drop a new file
into `files/`.

### Time Sources

`files/external.sources` sets the public [NTP Pool](https://www.ntppool.org/) as the upstream source:

```
pool pool.ntp.org iburst
server 0.pool.ntp.org iburst
server 1.pool.ntp.org iburst
server 2.pool.ntp.org iburst
server 3.pool.ntp.org iburst
```

To use other servers, edit this file or add more `*.sources` files.

### Client IP-Ranges

`files/allow.conf` lets clients from these ranges use the host as an NTP server:

| Range            | Purpose                              |
|------------------|--------------------------------------|
| `10.0.0.0/8`     | Private network (RFC 1918)           |
| `172.16.0.0/12`  | Private network (RFC 1918)           |
| `192.168.0.0/16` | Private network (RFC 1918)           |
| `169.254.0.0/16` | IPv4 link-local (RFC 3927)           |

Requests from other addresses are refused. To change the allowed clients, edit
the `allow` lines. To turn off the server function, delete `allow.conf`.

> **Note:** When you remove a file from `files/`, the role does not delete the
> copy that is already on the host. You have to remove it there by hand.
