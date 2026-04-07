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
Развёртывание на сервер: SSH под `debian` с ключом из [inventory.ini](inventory.ini) или [inventory-localdomain.ini](inventory-localdomain.ini). В инвентаре хост назван `ldap`, чтобы совпадало с обычным входом из WSL: `ssh debian@ldap` (для подключения задан `ansible_host` — IP сервера). Переменные LDAP — в [group_vars/ldap_servers.yml](group_vars/ldap_servers.yml) (включая служебную учётку `dummy`, нужную роли для `groupOfUniqueNames`). Плейбук [playbooks/localdomain.yml](playbooks/localdomain.yml) подключает роль **по пути** к корню репозитория — так команда работает и когда Ansible не читает `ansible.cfg` (типично WSL и клон на `/mnt/c/`).

Из корня репозитория явно укажите инвентарь:

```bash
ansible-playbook -i inventory.ini playbooks/localdomain.yml
```

Другой файл инвентаря или ключ:

```bash
ansible-playbook -i inventory-localdomain.ini playbooks/localdomain.yml
ANSIBLE_PRIVATE_KEY_FILE=~/.ssh/other_key ansible-playbook -i inventory.ini playbooks/localdomain.yml
```

Если `ansible.cfg` подхватывается (каталог не «world-writable»), можно опустить `-i`, если в конфиге задан `inventory`. При предупреждении про игнор `ansible.cfg` используйте `-i`, как выше, либо клонируйте репозиторий в обычную ФС Linux (например `$HOME` в WSL).

Переопределить пароль администратора LDAP:

```bash
ansible-playbook -i inventory.ini playbooks/localdomain.yml -e ldap_bind_pw=yoursecret
```

Если у `debian` на сервере sudo с паролем:

```bash
ansible-playbook -i inventory.ini playbooks/localdomain.yml --ask-become-pass
```

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
