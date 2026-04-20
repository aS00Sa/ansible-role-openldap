OpenLDAP Ansible Role
=====================
*Русский: [README.ru.md](README.ru.md)*

This role installs openldap and loads some example data. Intended for development and tests purposes, not to be used as production server

Requirements
------------
None

Role Variables
--------------
This role requires the following variables to be defined elsewhere in the playbook that uses it:
```yaml
ldap_basedn:       dc=mydomain,dc=net         # Base DN
ldap_server_uri:   ldap://localhost:389       # LDAP server URI
ldap_bind_pw:      secret                     # bind password
ldap_debconf_domain: mydomain.net             # optional (Debian): slapd debconf DNS domain for noninteractive install
openldap_firewall_open_ldap_port: true       # optional (Debian): allow TCP 389 in UFW when UFW is active
```

The role also expects a **dummy** LDAP user (uid `dummy`) in `ldap_users` so `groupOfUniqueNames` can be created; it is removed from group membership at the end of the play. The example `group_vars/ldap_servers.yml` includes this account.

Additionally, to load users and groups, you should define the following structure (example aligned with the localdomain demo: `debian`, `admin`, `user`, `developer`, `manager`):

```yaml
ldap_users:
  debian:
    cn: debian
    givenname: debian
    sn: user
    mail: debian@localdomain
    userpassword: debian
  admin:
    cn: admin
    givenname: admin
    sn: user
    mail: admin@localdomain
    userpassword: admin
  user:
    cn: user
    givenname: user
    sn: user
    mail: user@localdomain
    userpassword: user
  developer:
    cn: developer
    givenname: developer
    sn: user
    mail: developer@localdomain
    userpassword: developer
  manager:
    cn: manager
    givenname: manager
    sn: user
    mail: manager@localdomain
    userpassword: manager

ldap_groups:
  - name: devops
    members:
      - debian
      - developer
  - name: admin
    members:
      - admin
      - manager
  - name: users
    members:
      - user
```

Dependencies
------------
None

Example Playbook
----------------
Register the role in `requirements.yml`. The `src` value is Galaxy-style `namespace.role_name`; replace `localdomain` with your own organisation or username when publishing or vendoring the role:

```yaml
- src: localdomain.openldap-ansible-role
  name: openldap
```
Include it in your playbooks:
```yaml
- hosts: servers
  roles:
  - openldap
```

Local test install (localdomain)
--------------------------------
Example deploy: the inventory host is `ldap` (same idea as `ssh debian@ldap`); set `ansible_host` in [inventory-dev.ini](inventory-dev.ini) (or [inventory.ini](inventory.ini) / [inventory-localdomain.ini](inventory-localdomain.ini)). LDAP variables are in [group_vars/ldap_servers.yml](group_vars/ldap_servers.yml) (includes a technical `dummy` user for `groupOfUniqueNames`). Main play: [playbooks/install.yml](playbooks/install.yml) (loads the role by path from the repo root). [playbooks/localdomain.yml](playbooks/localdomain.yml) only imports `install.yml`.

From the repository root, typical WSL/Linux run with explicit config, user, key, verbose log, and tee (put `-vvv` **before** the playbook path so it is not mistaken for an extra playbook argument):

```bash
ANSIBLE_CONFIG="$PWD/ansible.cfg" ansible-playbook -vvv -i inventory-dev.ini -u debian --private-key ~/.ssh/id_rsa playbooks/install.yml 2>&1 | tee "ldap-inventory-localdomain-$(date +%Y%m%d-%H%M).log"
```

Shorter (inventory may set `ansible_user` / key):

```bash
ansible-playbook -i inventory.ini playbooks/install.yml
```

Another inventory or key:

```bash
ansible-playbook -i inventory-localdomain.ini playbooks/install.yml
ANSIBLE_PRIVATE_KEY_FILE=~/.ssh/other_key ansible-playbook -i inventory.ini playbooks/install.yml
```

`ANSIBLE_CONFIG="$PWD/ansible.cfg"` avoids Ansible ignoring `ansible.cfg` when the repo sits on a world-writable mount (e.g. `/mnt/c/` under WSL).

Override LDAP admin password:

```bash
ANSIBLE_CONFIG="$PWD/ansible.cfg" ansible-playbook -i inventory-dev.ini -u debian --private-key ~/.ssh/id_rsa playbooks/install.yml -e ldap_bind_pw=yoursecret
```

If `debian` uses sudo with a password on the target host, add `--ask-become-pass` to any of the commands above.

On Debian/Ubuntu targets with **UFW** enabled, the role opens **TCP 389** when `openldap_firewall_open_ldap_port` is true (default in [defaults/main.yml](defaults/main.yml)).

Optional: Apache Directory Studio (Windows and Linux)
-------------------------------------------------------
[Apache Directory Studio](https://directory.apache.org/studio/) is a free LDAP browser and editor (Eclipse-based); the same installer workflow applies on Windows and on Linux. This role deploys **OpenLDAP**; use the client to connect to your server’s hostname or IP on port **389** (or **636** with LDAPS if you add TLS on the server).

Download Studio, open **LDAP** → **New Connection**, then set fields to match your Ansible variables (example for the localdomain demo in `group_vars/ldap_servers.yml`):

| Field | Example value |
| ----- | ------------- |
| Hostname | `ldap` (same as `ssh debian@ldap`) or `192.168.x.x` |
| Port | `389` |
| Encryption method | `No encryption` (until TLS is configured on slapd) |
| Bind DN | `dc=localdomain` (same as `ldap_basedn`) |
| Bind password | value of `ldap_bind_pw` |

This role does not install Apache Directory Studio; install it on your workstation from the link above.

License
-------

BSD
