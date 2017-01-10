## Роли

В прошлом разделе мы разобрались с использованием include.
Это был первый способ разбития playbook на части.
В этом разделе мы рассмотрим второй способ - роли.

Роли это способ логического разбития файлов Ansible.
По сути роли это просто автоматизация выражений include, которая основана на определенной файловой структуре.
То есть, нам не нужно будет явно указывать полные пути к файлам с задачами или сценариями, а достаточно лишь соблюдать определенную структуру файлов.

Но, засчет этого, работать с Ansible намного удобней.
И у нас рождается модульная структура, которая разбита на роли, например, на основе функциональности.

Для того, чтобы мы могли использовать роли, нужно соблюдать определенную структуру каталогов:
```
├── all_roles.yml
├── cfg_security.yml
├── cfg_ospf.yml
|
└── roles
    ├── ospf
    │   ├── files
    │   ├── templates
    │   ├── tasks
    │   ├── handlers
    │   ├── vars
    │   ├── defaults
    │   └── meta
    └── security
        ├── files
        ├── templates
        ├── tasks
        ├── handlers
        ├── vars
        ├── defaults
        └── meta
```

Первые три файла - это playbook. Они используют созданные роли.

Например, playbook all_roles.yml выглядит так:
```yml
---

- name: Roles config
  hosts: cisco-routers
  gather_facts: false
  connection: local
  roles:
    - security
    - ospf
```

Остальные файлы: инвентарный, конфигурационный файл Ansible и каталоги с переменными, находятся в тех же местах (в том же каталоге, что и playbook).


Все роли, по умолчанию, должны быть определены в каталоге roles:
* Каталоги следующего уровня определяют названия ролей
 * В примере выше, созданы две роли: ospf и security
* Внутри каждой роли могут быть указанные каталоги.
 * Как минимум, понадобится каталог tasks, чтобы описать задачи, а все остальные каталоги опциональны.
 * Внутри каталогов tasks, handlers, vars, defaults, meta автоматически считывается всё, что находится в файле main.yml
   * если в этих каталогах есть другие файлы, их надо добавлять через include
 * Внутри роли, на файлы в каталогах files, templates, tasks можно ссылаться не указывая путь к ним (достаточно указать имя файла)

Каталоги внутри роли:
* tasks - если в этом каталоге существует файл main.yml, все задачи, которые в нем указаны, будут добавлены в сценарий
 * если в каталоге tasks есть файл с задачами с другим названием, его можно добавить в роль через include, при этом не нужно указывать путь к файлу
* handlers - если в этом каталоге существует файл main.yml, все handlers, которые в нем указаны, будут добавлены в сценарий
* vars - если в этом каталоге существует файл main.yml, все переменные, которые в нем указаны, будут добавлены в сценарий
* defaults - каталог, в котором указываются значения по умолчанию для переменных. Эти значения имеют самый низкий приоритет, поэтому их легко перебить, определив переменную в другом месте. Если в этом каталоге существует файл main.yml, все переменные, которые в нем указаны, будут добавлены в сценарий
* meta - каталог, в котором указаны зависимости роли. Если в этом каталоге существует файл main.yml, все роли, которые в нем указаны, будут добавлены в список ролей
* files - каталог, в котором могут находиться различные файлы. Например, файл конфигурации
* templates - каталог для шаблонов. Если нужно указать шаблон из этого каталога, достаточно указать имя, без пути к файлу

### Пример использования ролей

Рассмотрим пример использования ролей.

Структура каталога 8_playbook_roles выглядит таким образом:
```
├── ansible.cfg
├── myhosts
|
├── all_roles.yml
├── cfg_initial.yml
├── cfg_ospf.yml
|
├── group_vars
│   ├── all.yml
│   ├── cisco-routers.yml
│   └── cisco-switches.yml
├── host_vars
│   ├── 192.168.100.1
│   ├── 192.168.100.100
│   ├── 192.168.100.2
│   └── 192.168.100.3
|
└── roles
    ├── ospf
    │   ├── handlers
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       └── ospf.j2
    ├── security
    │   └── tasks
    │       └── main.yml
    └── usability
        └── tasks
            └── main.yml
```

Файл конфигурации Ansible, инвентарный файл и каталоги с переменными остались без изменений.

Добавлен каталог roles, в котором находятся три роли: usability, security и ospf.

Для ролей usability и security создан только каталог tasks и в нем находится только один файл: main.yml.

Содержимое файла roles/usability/tasks/main.yml:
```yml
---

- name: Global usability config
  ios_config:
    lines:
      - no ip domain lookup
    provider: "{{ cli }}"

- name: Configure vty usability features
  ios_config:
    parents:
      - line vty 0 4
    lines:
      - exec-timeout 30 0
      - logging synchronous
      - history size 100
    provider: "{{ cli }}"
```

В нем находятся две задачи. Они достаточно простые и должны быть полностью понятны.

Обратите внимание, что в файле определяются только задачи.
К каким хостам они будут применяться, будет определять playbook, который будет использовать роль.

Содержимое файла roles/security/tasks/main.yml также должно быть понятно:
```yml
---

- name: Global security config
  ios_config:
    lines:
      - service password-encryption
      - no ip http server
      - no ip http secure-server
    provider: "{{ cli }}"

- name: Configure vty security features
  ios_config:
    parents:
      - line vty 0 4
    lines:
      - transport input ssh
    provider: "{{ cli }}"
```

> **Note** Несмотря на то, что функционал достаточно простой и общий, мы разделили его на две роли. Такое разделение позволяет более четко описать цель роли.

Теперь посмотрим как будет выглядеть playbook, который использует обе роли (файл cfg_initial.yml):
```
---

- name: Initial config
  hosts: cisco-routers
  gather_facts: false
  connection: local
  roles:
    - usability
    - security
```

Теперь запустим playbook (предварительно на маршрутизаторах сделаны изменения):
```
$ ansible-playbook cfg_initial.yml
```

![cfg_initial](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/cfg_initial.png)

Обратите внимание, что теперь, когда задачи выполняются, перед именем задачи написано имя роли:
```
TASK [usability : Configure vty usability features]
```

Теперь разберемся с ролью ospf.
В этой роли используется несколько файлов.

Файл roles/ospf/tasks/main.yml описывает задачи:
```yml
---

- name: Collect facts
  ios_facts:
    gather_subset:
      - "!hardware"
    provider: "{{ cli }}"

- name: Set fact ospf_networks
  set_fact:
    current_ospf_networks: "{{ ansible_net_config | regex_findall('network (.*) area 0') }}"

- name: Show var current_ospf_networks
  debug: var=current_ospf_networks

- name: Config OSPF
  ios_config:
    src: ospf.j2
    provider: "{{ cli }}"
  notify: save config

- name: Write OSPF cfg in variable
  ios_command:
    commands:
      - sh run | s ^router ospf
    provider: "{{ cli }}"
  register: ospf_cfg

- name: Show OSPF cfg
  debug: var=ospf_cfg.stdout_lines
```

Разберемся с содержимым файла:
* Сначала мы собираем все факты об устройствах, кроме hardware.
* Затем вручную устанавливаем факт current_ospf_networks
 * фильтруем конфигурацию устройства и находим все строки с командами ```network ... area 0```. Всё, что находится между указанными словами, запоминается.
 * в итоге, мы получим список с командами
* Следующая задача показывает содержимое переменной current_ospf_networks
* Задача "Config OSPF" настраивает OSPF по шаблону ospf.j2
 * если изменения были, выполняется handler save config
* Последующие задачи выполняют команду ```sh run | s ^router ospf``` и отображают содержимое

Файл roles/ospf/handlers/main.yml:
```yml
- name: save config
  ios_command:
    commands:
      - write
    provider: "{{ cli }}"
```

Файл roles/ospf/templates/ospf.j2:
```
router ospf 1
 router-id {{ mgmnt_ip }}
 ispf
 auto-cost reference-bandwidth 10000
{% for ip in ansible_net_all_ipv4_addresses %}
 network {{ ip }} 0.0.0.0 area 0
{% endfor %}
{% for network in current_ospf_networks %}
 {% if network.split()[0] not in ansible_net_all_ipv4_addresses %}
   no network {{ network }} area 0
 {% endif %}
{% endfor %}
```

В шаблоне мы используем переменные:
* mgmnt_ip - определена в соответствующем файле каталога host_vars/
* ansible_net_all_ipv4_addresses - эта переменная содержит список всех IP-адресов устройства. Это факт, который обнаруживается благодаря модулю ios_facts
* current_ospf_networks - факт, который мы создали вручную

Получается, что в шаблоне настраиваются команды network, на основе IP-адресов устройства, а затем удаляются лишние команды network.

Проверим работу роли на примере такого playbook cfg_ospf.yml:
```yml
---

- name: Configure OSPF
  hosts: 192.168.100.1
  gather_facts: false
  connection: local
  roles:
    - ospf
```

Начальная конфигурация R1 такая (две лишних команды network):
```
R1#sh run | s ^router ospf
router ospf 1
 router-id 10.0.0.1
 ispf
 auto-cost reference-bandwidth 10000
 network 10.1.1.1 0.0.0.0 area 0
 network 10.10.1.1 0.0.0.0 area 0
 network 192.168.100.1 0.0.0.0 area 0
 network 192.168.200.1 0.0.0.0 area 0

R1#show ip int bri | exc unass
Interface        IP-Address      OK? Method Status      Protocol
Ethernet0/0      192.168.100.1   YES NVRAM  up          up
Ethernet0/1      192.168.200.1   YES NVRAM  up          up
```

Теперь запустим playbook и посмотрим удалятся ли две лишние команды:
```
$ ansible-playbook cfg_ospf.yml
```

![cfg_ospf](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/cfg_ospf.png)


Обратите внимание, что до выполнения конфигурации было 4 команды network (мы их видим по содержимому переменной current_ospf_networks):
```
    "current_ospf_networks": [
        "10.1.1.1 0.0.0.0",
        "10.10.1.1 0.0.0.0",
        "192.168.100.1 0.0.0.0",
        "192.168.200.1 0.0.0.0"
    ]
```

А после конфигурации, осталось две команды network:
```
    "ospf_cfg.stdout_lines": [
        [
            "router ospf 1",
            " router-id 10.0.0.1",
            " ispf",
            " auto-cost reference-bandwidth 10000",
            " network 192.168.100.1 0.0.0.0 area 0",
            " network 192.168.200.1 0.0.0.0 area 0"
        ]
    ]
```

> **Note** Этот пример не идеален. Например, подразумевается, что все интерфейсы находятся в зоне 0. Но его достаточно, чтобы понять как использовать роли.

Скорее всего, в реальной жизни вы уберете задачи, которые отображают содержимое переменных. Но, для того чтобы лучше разобраться с тем, что делает роль, они полезны.

На этом мы заканчиваем раздел. О других возможностях использования ролей вы можете почитать в [документации, в разделе роли](http://docs.ansible.com/ansible/playbooks_roles.html#roles).
