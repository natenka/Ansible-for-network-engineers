## lines (commands)

Самый простой способ использовать модуль ios_config - отправлять команды глобального конфигурационного режима с параметром lines (для параметра lines есть alias commands, то есть, можно вместо lines писать commands).

Посмотрим на пример playbook 6_ios_config_lines.yml:
```yml
---

- name: Run cfg commands on routers
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: Config password encryption
      ios_config:
        lines:
          - service password-encryption
        provider: "{{ cli }}"
```

Мы, по-прежнему используем переменную cli, которую указали в файле group_vars/all.yml, для того, чтобы указать параметры подключения.

Если мы запустим playbook, то получим такой результат:
```
$ ansible-playbook 6_ios_config_lines.yml
```

![6_ios_config_lines](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6_ios_config_lines.png)

Какие команды Ansible выполняет:
* terminal length 0
* enable
* show running-config - чтобы проверить есть ли эта настройка на устройстве. Если команда есть, задача выполняться не будет. Если команды нет, задача выполнится
* если команды, которая указана в задаче нет в конфигурации:
 * configure terminal
 * service password-encryption
 * end

Так как модуль каждый раз проверяет конфигурацию, прежде чем применит команду, модуль идемпотентен.
То есть, если мы ещё раз запустим playbook, а команда уже настроена, изменения не будут выполнены:
```
$ ansible-playbook 6_ios_config_lines.yml
```

![6_ios_config_lines](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6_ios_config_lines_2.png)

> **Caution** Обязательно пишите команды полностью, а не сокращенно. И обращайте внимание, что, для некоторых команд, IOS сам добавляет параметры. Если писать команду не в том виде, в котором она реально видна в конфигурационном файле, модуль не будет идемпотентен. Он будет всё время считать, что команды нет и вносить изменения каждый раз. 

Параметр lines позволяет отправлять и несколько команд (playbook 6_ios_config_mult_lines.yml):
```
---

- name: Run cfg commands on routers
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: Send config commands
      ios_config:
        lines:
          - service password-encryption
          - no ip http server
          - no ip http secure-server
          - no ip domain lookup
        provider: "{{ cli }}"
```

Результат выполнения:
```
$ ansible-playbook 6_ios_config_mult_lines.yml
```

![6_ios_config_mult_lines](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6_ios_config_mult_lines.png)
