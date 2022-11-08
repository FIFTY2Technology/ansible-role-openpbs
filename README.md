# Description
This role installs and configures OpenPBS on computenodes, servernodes, or both. It follows the `INSTALL` instructions from the [official GitHub repository](https://github.com/openpbs/openpbs/blob/master/INSTALL).

**Note**: When applied to a node with a running simulation and a configuration change takes place, OpenPBS is restarted and therefore also restarts the simulation. To be on the safe side, only run on idling nodes.

# Requirements
* All nodes must be able to use SSH between each other (n-to-n) (includes OpenPBS server *and* nodes) with the desired MPI-user.
* Inventory groups should be named `pbsheadnode` (for headnode) and `mpinodes` (for compute nodes). Deviant group names must be set in default variables `openpbs_server_hostname`, `openpbs_server`, `openpbs_mom`.
* Supported Linux distribution include Debian 10, Ubuntu 18.04 and CentOS 8.
* Working DNS name resolution. Includes resolution of FQDNs and non-FQDNs:
  * Make sure your DNS servers resolve hostnames correctly
  * Make sure `/etc/resolv.conf` includes both correct `search` and `domain` entries
  * Hostnames must be in non-FQDN format (`foo` instead of `foo.example.org`)
* RedHat-based distributions need PowerTools/CRB repository enabled.

# Role Variables
All variables which can be overridden are stored in defaults/main.yml file as well as in table below.

| Name | Default value | Description |
| ------ | ------ | ----- |
| `openpbs_user` | openpbs | Username to own the sourcecode directory. User will get `sudo` privileges. |
| `openpbs_group` | `"{{ openpbs_user }}"` | Groupname to own the sourcecode directory |
| `openpbs_home_dir` | `"/home/{{ openpbs_user }}"` | Directory to download the sourcecode into |
| `openpbs_version` | latest | OpenPBS version to install. Must be 'latest' or match a tag name in OpenPBS (e.g. `v20.0.1`). See GitHub repostiory: https://github.com/openpbs/openpbs/tags |
| `openpbs_prefix` | /opt/pbs | Installation prefix |
| `openpbs_server_hostname` | `"{{ groups['pbsheadnode'][0] }}"` | Hostname of the OpenPBS server (headnode). Default is based on inventory group membership. |
| `openpbs_queue` | `- name: myqueue`<br>`  type: execution`<br>`  enabled: true`<br>`  started: true` | List of queues to create. Only the shown basic configuration options are supported. |
| `openpbs_server` | `"{{ ( inventory_hostname in groups['pbsheadnode'] ) and not`<br>`  ( inventory_hostname in groups['mpinodes'] ) }}"` | Decide if a host is a PBS MOM or a PBS server, based on its group memberships. Adapt group names where necessary, logic must not be changed. |
| `openpbs_mom` | `"{{ ( inventory_hostname in groups['mpinodes'] ) and not`<br>`  ( inventory_hostname in groups['pbsheadnode'] ) }}"` | Decide if a host is a PBS MOM or a PBS server, based on its group memberships. Adapt group names where necessary, logic not be changed. |


# Dependencies
None.

# Example Playbook
Inventory used for this example:
```
all:
  children:
    mpinodes:
      hosts:
        ubuntumom.example.com:
        centosmom.example.com:
        debianmom.example.com:
    pbsheadnode:
      hosts:
        pbsserver.example.com:
```

```
- hosts: mpinodes,pbsheadnode
  gather_facts: true
  become: true

  roles:
    - role: openpbs
```
