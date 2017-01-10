## after

Параметр __after__ указывает какие команды выполнить после команд в списке lines (или commands).

Команды, которые указаны в параметре after:
* выполняются только если должны быть внесены изменения.
* при этом они будут выполнены, независимо от того есть они в конфигурации или нет.

Параметр after очень полезен в ситуациях, когда нам нужно выполнить команду, которая не сохраняется в конфигурации.
Например, команда no shutdown не сохраняется в конфигурации маршрутизатора.
И, если бы мы написали её в списке lines, то изменения вносились бы каждый раз, при выполнении playbook. 

Но, если мы напишем команду no shutdown в списке after, то только в том случае, если нужно вносить изменения (согласно списка lines), будет применена и команда no shutdown.

Пример использования параметра after в playbook 6f_ios_config_after.yml:
```yml
---

- name: Run cfg commands on router
  hosts: 192.168.100.1
  gather_facts: false
  connection: local

  tasks:

    - name: Config interface
      ios_config:
        parents:
          - interface Ethernet0/3
        lines:
          - ip address 192.168.230.1 255.255.255.0
        after:
          - no shutdown
        provider: "{{ cli }}"
```

Первый запуск playbook, с внесением изменений:
```
$ ansible-playbook 6f_ios_config_after.yml -v
```
![6f_ios_config_after.png](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6f_ios_config_after.png)


Второй запуск playbook (изменений нет, поэтому команда no shutdown не выполняется):
```
$ ansible-playbook 6f_ios_config_after.yml -v
```
![6f_ios_config_after_no_change](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6f_ios_config_after_no_change.png)


Рассмотрим ещё один пример использования after.
Сохраним, с помощью after, конфигурацию устройства (playbook 6f_ios_config_after_save.yml):
```yml
---

- name: Run cfg commands on routers
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: Config line vty
      ios_config:
        parents:
          - line vty 0 4
        lines:
          - login local
          - transport input ssh
        after:
          - end
          - write
        provider: "{{ cli }}"
```

Результат выполнения playbook (изменения только на маршрутизаторе 192.168.100.1):
```
$ ansible-playbook 6f_ios_config_after_save.yml -v
```
![6f_ios_config_after_save](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6f_ios_config_after_save.png)


