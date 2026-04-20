Роль Ansible для OpenLDAP
=========================
Роль устанавливает OpenLDAP и загружает пример данных. Предназначена для разработки и тестов, не для production.

*English: [README.md](README.md)*

Требования
----------
Нет.

Переменные роли
---------------
Их нужно задать в плейбуке, который подключает роль:

```yaml
ldap_basedn:       dc=mydomain,dc=net         # базовый DN
ldap_server_uri:   ldap://localhost:389       # URI сервера LDAP
ldap_bind_pw:      secret                     # пароль привязки (администратора каталога)
ldap_debconf_domain: mydomain.net             # необязательно (Debian): DNS-домен для debconf slapd при установке без вопросов
openldap_firewall_open_ldap_port: true        # необязательно (Debian): открыть 389/tcp в UFW при активном UFW
```

Роли нужна служебная учётка LDAP **dummy** (uid `dummy`) в `ldap_users` для создания `groupOfUniqueNames`; в конце плейбука она убирается из состава групп. Пример с ней — в `group_vars/ldap_servers.yml`.

Для загрузки пользователей и групп задайте структуру (пример в духе демо `localdomain`: пользователи `debian`, `admin`, `user`, `developer`, `manager`):

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

Зависимости
-----------
Нет.

Пример плейбука
---------------
Регистрация роли в `requirements.yml`. Значение `src` в стиле Galaxy — `namespace.role_name`; при публикации или подключении роли замените `localdomain` на свой организационный аккаунт или имя пользователя:

```yaml
- src: localdomain.openldap-ansible-role
  name: openldap
```

Подключение в плейбуках:

```yaml
- hosts: servers
  roles:
  - openldap
```

Локальный тест (localdomain)
-----------------------------
Развёртывание: в инвентаре хост `ldap` (как `ssh debian@ldap`), `ansible_host` — IP в [inventory-dev.ini](inventory-dev.ini) (или [inventory.ini](inventory.ini) / [inventory-localdomain.ini](inventory-localdomain.ini)). Переменные LDAP — [group_vars/ldap_servers.yml](group_vars/ldap_servers.yml) (включая служебную `dummy` для `groupOfUniqueNames`). Основной плейбук: [playbooks/install.yml](playbooks/install.yml) (роль по пути из корня репозитория). [playbooks/localdomain.yml](playbooks/localdomain.yml) только подключает `install.yml`.

Типичный запуск из корня репозитория в WSL/Linux: явный `ansible.cfg`, пользователь, ключ, подробный лог и `tee`. Флаг `-vvv` ставьте **до** пути к плейбуку, иначе он может восприниматься как лишний аргумент:

```bash
ANSIBLE_CONFIG="$PWD/ansible.cfg" ansible-playbook -vvv -i inventory-dev.ini -u debian --private-key ~/.ssh/id_rsa playbooks/install.yml 2>&1 | tee "ldap-inventory-localdomain-$(date +%Y%m%d-%H%M).log"
```

Короче (если в инвентаре заданы `ansible_user` и ключ):

```bash
ansible-playbook -i inventory.ini playbooks/install.yml
```

Другой инвентарь или ключ:

```bash
ansible-playbook -i inventory-localdomain.ini playbooks/install.yml
ANSIBLE_PRIVATE_KEY_FILE=~/.ssh/other_key ansible-playbook -i inventory.ini playbooks/install.yml
```

`ANSIBLE_CONFIG="$PWD/ansible.cfg"` помогает, когда Ansible игнорирует `ansible.cfg` из каталога на world-writable томе (например `/mnt/c/` в WSL).

Переопределить пароль администратора LDAP:

```bash
ANSIBLE_CONFIG="$PWD/ansible.cfg" ansible-playbook -i inventory-dev.ini -u debian --private-key ~/.ssh/id_rsa playbooks/install.yml -e ldap_bind_pw=yoursecret
```

Если у `debian` на сервере sudo с паролем — добавьте `--ask-become-pass` к любой из команд выше.

На Debian/Ubuntu при **активном UFW** роль открывает **TCP 389**, если включено `openldap_firewall_open_ldap_port` (по умолчанию — см. [defaults/main.yml](defaults/main.yml)).

По желанию: Apache Directory Studio (Windows и Linux)
-------------------------------------------------------
[Apache Directory Studio](https://directory.apache.org/studio/) — бесплатный просмотр и редактирование LDAP (на базе Eclipse); установка одинакова на Windows и Linux. Роль развёртывает **OpenLDAP**; в клиенте укажите имя хоста или IP сервера и порт **389** (или **636** с LDAPS, если настроите TLS на сервере).

Скачайте Studio, откройте **LDAP** → **New Connection** и заполните поля по вашим переменным Ansible (пример для демо localdomain в `group_vars/ldap_servers.yml`):

| Поле | Пример значения |
| ---- | --------------- |
| Hostname | `ldap` (как в `ssh debian@ldap`) или `192.168.x.x` |
| Port | `389` |
| Encryption method | `No encryption` (пока на slapd не настроен TLS) |
| Bind DN | `dc=localdomain` (то же, что `ldap_basedn`) |
| Bind password | значение `ldap_bind_pw` |

Роль не устанавливает Apache Directory Studio; ставьте клиент на рабочую машину по ссылке выше.

Лицензия
--------

BSD
