## Модуль ios_command

Модуль ios_command - отправляет команду на устройство под управлением IOS и возвращает результат выполнения команды.

> **Note** Модуль ios_command не поддерживает отправку команд в конфигурационном режиме.
> Для этого используется отдельный модуль - ios_config.

Когда мы отправляем команду на устройство, модуль самостоятельно аутентифицируется по SSH, переходит в режим enable и дает команду ```terminal length 0```, чтобы вывод команд show отражался полностью, а не постранично.

Посмотрим на простой пример использования модуля ios_command (playbook 4_ios_command.yml):
```
---

- name: Run show commands on routers
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: run sh ip int br
      ios_command:
        commands: show ip int br
        provider: "{{ cli }}"
      register: sh_ip_int_br_result

    - name: Debug registered var
      debug: var=sh_ip_int_br_result.stdout_lines
```

Модуль ios_command ожидает параметры:
* commands - список команд, которые нужно отправить на устройство
* provider - словарь с параметрами подключения
 * в нашем случае, он указан в файле group_vars/all.yml

> **Caution** Обратите внимание, что параметр register находится на одном уровне с именем задачи и модулем, а не на уровне параметров модуля ios_command.

Попробуем запустить playbook:
```
$ ansible-playbook 2_ios_command.yml
```

![ios_command](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/2_ios_command.png)


В отличии от использования модуля raw, когда мы используем модуль ios_command, playbook не указывает, что были выполнены изменения.


### Выполнение нескольких команд

Модуль ios_command позволяет выполнять несколько команд.
Попробуем выполнить несколько команд и получить их вывод.

Playbook 4a_ios_command.yml:
```
---

- name: Run show commands on routers
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: run show commands
      ios_command:
        commands:
          - show ip int br
          - sh ip route
        provider: "{{ cli }}"
      register: show_result

    - name: Debug registered var
      debug: var=show_result.stdout_lines
```

Теперь мы указываем две команды, в модуле ios_command, поэтому синтаксис должен быть немного другим - команды должны быть указаны, как список в формате YAML.


Посмотрим на результат выполнения playbook (вывод сокращен):
```
$ ansible-playbook 2a_ios_command.yml
```

![ios_command](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/2a_ios_command.png)

Обе команды выполнились на всех устройствах.

Если мы сохраняем результат выполнения команд, как в этом playbook, то вывод будет находится в переменных stdout и stdout_lines в отдельных списках.
То есть, в этих переменных теперь будет находится список, внутри которого находятся списки с выводом команд, по порядку описания их в задаче.

Например, мы могли бы вывести вывод только первой команды, указав:
```
    - name: Debug registered var
      debug: var=show_result.stdout_lines[0]
```

### Обработка ошибок

В модуле встроено распознание ошибок.
Поэтому, если команда выполнена с ошибкой, модуль отобразит, что возникла ошибка.

Например, если сделать ошибку в команде, и запустить playbook еще раз
```
$ ansible-playbook 2_ios_command.yml
```

![ios_command](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/2_ios_command-fail.png)

То есть, если задача отработала, значит ошибок при выполнении не было.

Аналогичным образом модуль обнаруживает ошибки:
* Ambiguous command
* Incomplete command


