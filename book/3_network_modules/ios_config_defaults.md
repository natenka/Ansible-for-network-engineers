## defaults

Параметр __defaults__ указывает нужно ли собираться всю информацию с устройства, в том числе и значения по умолчанию.
Если включить этот параметр, то модуль будет собирать текущую кофигурацию с помощью команды sh run all.
По умолчанию этот параметр отключен и конфигурация проверяется командой sh run.

Этот параметр полезен в том случае, если мы указываем в настройках команду, которая не видна в конфигурации.
Например, такое может быть, когда мы указали параметр, который и так используется по умолчанию.

Если мы не будем использовать параметр defaults, и укажем команду со значением, по умолчанию (например, в playbook ниже мы указываем команду ip mtu 1500), то каждый раз, когда мы запускаем playbook, будут вноситься изменения.
Присходит это потому, что Ansible каждый раз вначале проверяет наличие команд в соответствующем режиме.
Если команд нет, то соответствующая задача выполняется.


В такой варианте playbook 6e_ios_config_defaults.yml каждый раз будут вноситься изменения (попробуйте самостоятельно):
```yml
---

- name: Run cfg commands on routers
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: Config interface
      ios_config:
        parents:
          - interface Ethernet0/2
        lines:
          - ip address 192.168.200.1 255.255.255.0
          - ip mtu 1500
        provider: "{{ cli }}"
```

Если же мы добавим параметр defaults: yes, изменения уже не будут внесены, если не хватало только команды ip mtu 1500 (playbook 6e_ios_config_defaults.yml):
```
---

- name: Run cfg commands on routers
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: Config interface
      ios_config:
        parents:
          - interface Ethernet0/2
        lines:
          - ip address 192.168.200.1 255.255.255.0
          - ip mtu 1500
        defaults: yes
        provider: "{{ cli }}"
```

Запуск playbook:
```
$ ansible-playbook 6e_ios_config_defaults.yml
```

![6e_ios_config_default](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6e_ios_config_defaults.png)
