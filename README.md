OpenLDAP Ansible Role
=====================
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
```

Additionally, to load users and groups, you should define the following structure
```yaml
ldap_users:
  user_id1:
    cn: Name1 Surname1
    givenname: Name1
    sn: Surname1
    mail: userid1@mydomain.net
    userpassword: password
  user_id2:
    cn: Name2 Surname2
    givenname: Name2
    sn: Surname2
    mail: userid2@mydomain.net
    userpassword: password
ldap_groups:
  - name: group1
    members:
      - user_id1
  - name: group2
    members:
      - user_id1
      - user_id2
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
Example deploy uses SSH as user `debian` and a key (default `~/.ssh/id_ed25519` in [inventory-localdomain.ini](inventory-localdomain.ini)), [ansible.cfg](ansible.cfg) (same layout as `ansible-role-nexus3-oss`: `roles_path`, inventory, SSH multiplexing), and LDAP data in [group_vars/ldap_servers.yml](group_vars/ldap_servers.yml). The play references the role by repository directory name `ansible-role-openldap`.

From the repository root (`inventory` is set in `ansible.cfg`):

```bash
ansible-playbook playbooks/localdomain.yml
```

Another inventory or SSH key:

```bash
ansible-playbook -i inventory-localdomain.ini playbooks/localdomain.yml
ANSIBLE_PRIVATE_KEY_FILE=~/.ssh/other_key ansible-playbook playbooks/localdomain.yml
```

Override LDAP admin password:

```bash
ansible-playbook playbooks/localdomain.yml -e ldap_bind_pw=yoursecret
```

If `debian` uses sudo with a password on the target host:

```bash
ansible-playbook playbooks/localdomain.yml --ask-become-pass
```

License
-------

BSD
