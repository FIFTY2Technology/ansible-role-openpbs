# Description
This role installs and configures OpenPBS on computenodes, servernodes, or both. It follows the `INSTALL` instructions from the [official GitHub repository](https://github.com/openpbs/openpbs/blob/master/INSTALL).

**Note**: When applied to a node with a running simulation and a configuration change takes place, OpenPBS is restarted and therefore also restarts the simulation. To be on the safe side, only run on idling nodes.

Molecule scenario is tested in Podman with Rockylinux 9 and OpenPBS version 23.06.06.

# Requirements
* All nodes must be able to use SSH between each other (n-to-n) (includes OpenPBS server *and* nodes) with the desired MPI-user.
* Inventory groups should be named `pbsheadnode` (for headnode) and `mpinodes` (for compute nodes). Deviant group names must be set in default variables `openpbs_server_hostname`, `openpbs_server`, `openpbs_mom` and in `templates/pbs.conf.j2`.
* Supported Linux distribution include Debian 11, Ubuntu 18.04 and Rockylinux 8/9.
* Working DNS name resolution. Includes resolution of FQDNs and non-FQDNs:
  * Make sure your DNS servers resolve hostnames correctly
  * Make sure `/etc/resolv.conf` includes both correct `search` and `domain` entries
  * Hostnames must be in non-FQDN format (`foo` instead of `foo.example.org`)
* RedHat-based distributions need PowerTools/CRB repository enabled.

# Role Variables
All variables which can be overridden are stored in defaults/main.yml file as well as in table below.

| Name | Default value | Description |
| ------ | ------ | ----- |
| `openpbs_version` | latest | OpenPBS version to install. Must be 'latest' or match a tag name in OpenPBS (e.g. `v20.0.1`). See GitHub repostiory: https://github.com/openpbs/openpbs/tags |
| `openpbs_install_dir` | /opt/pbs | Installation directory. Will also be added to your `$PATH`. |
| `openpbs_build_dir` | /opt/build_pbs | Build directory |
| `openpbs_pbs_conf_template` | pbs.conf.j2 | Path to the config file template for `/etc/pbs.conf` |
| `openpbs_mom_priv_config_template` | mom_priv_config.j2 | Path to the config file template for `/var/spool/pbs/mom_priv/config`
| `openpbs_server_hostname` | `"{{ groups['pbsheadnode'][0] }}"` | Hostname of the OpenPBS server (headnode). Default is based on inventory group membership. |
| `openpbs_server` | `"{{ ( inventory_hostname in groups['pbsheadnode'] ) and not`<br>`  ( inventory_hostname in groups['mpinodes'] ) }}"` | Decide if a host is a PBS MOM or a PBS server, based on its group memberships. Adapt group names where necessary, logic must not be changed. |
| `openpbs_mom` | `"{{ ( inventory_hostname in groups['mpinodes'] ) and not`<br>`  ( inventory_hostname in groups['pbsheadnode'] ) }}"` | Decide if a host is a PBS MOM or a PBS server, based on its group memberships. Adapt group names where necessary, logic not be changed. |
| `openpbs_dbuser_username` | pbsdata | User created for PBS-internal use, for accessing the database. Can be handy to have access to in case update breaks and you have to check the DB manually. |
| `openpbs_dbuser_password` | plzchangeme | Password for above account name |

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
