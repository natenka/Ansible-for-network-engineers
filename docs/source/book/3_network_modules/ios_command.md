{% raw %}
## Модуль ios_command

Модуль __ios_command__ - отправляет команду show на устройство под управлением IOS и возвращает результат выполнения команды.

> **Note** Модуль ios_command не поддерживает отправку команд в конфигурационном режиме. Для этого используется отдельный модуль - ios_config.

Перед отправкой самой команды, модуль:
* выполняет аутентификацию по SSH,
* переходит в режим enable
* выполняет команду ```terminal length 0```, чтобы вывод команд show отражался полностью, а не постранично.

Пример использования модуля ios_command (playbook 1_ios_command.yml):
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

Результат выполнения playbook:
```
$ ansible-playbook 1_ios_command.yml
```
{% endraw %}
![ios_command]({{ book.ansible_img_path }}2_ios_command.png)

{% raw %}
> В отличии от использования модуля raw, playbook не указывает, что были выполнены изменения.


### Выполнение нескольких команд

Модуль ios_command позволяет выполнять несколько команд.

Playbook 2_ios_command.yml выполняет несколько команд и получает их вывод:
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

В первой задаче указываются две команды, поэтому синтаксис должен быть немного другим - команды должны быть указаны как список, в формате YAML.


Результат выполнения playbook (вывод сокращен):
```
$ ansible-playbook 2_ios_command.yml
```
{% endraw %}
![ios_command]({{ book.ansible_img_path }}2a_ios_command.png)

Обе команды выполнились на всех устройствах.


Если модулю передаются несколько команд, результат выполнения команд находится в переменных stdout и stdout_lines в списке. Вывод будет в том порядке, в котором команды описаны в задаче.

Засчет этого, например, можно вывести результат выполнения первой команды, указав:
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

![ios_command]({{ book.ansible_img_path }}2_ios_command-fail.png)

Ansible обнаружил ошибку и возвращает сообщение ошибки.
В данном случае - 'Invalid input'.

Аналогичным образом модуль обнаруживает ошибки:
* Ambiguous command
* Incomplete command

