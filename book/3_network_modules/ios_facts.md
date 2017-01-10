## Модуль ios_facts

Модуль ios_facts - собирает информацию с устройств под управлением IOS.

Информация берется из таких команд:
* dir
* show version
* show memory statistics
* show interfaces
* show ipv6 interface
* show lldp
* show lldp neighbors detail
* show running-config

>**Note** Вы можете настроить [EEM applet](http://xgu.ru/wiki/EEM#.D0.9F.D1.80.D0.B8.D0.BC.D0.B5.D1.80.D1.8B_.D1.81.D0.BE.D0.B1.D1.8B.D1.82.D0.B8.D1.8F_cli), который будет генерировать лог сообщения о выполненных командах, чтобы видеть какие команды выполняет Ansible.

В модуле можно указывать какие параметры собирать - можно собирать всю информацию, а можно только подмножество.
По умолчанию, модуль собирает всю информацию, кроме конфигурационного файла.

Какую информацию собирать, указывается в параметре gather_subset.
Поддерживаются такие варианты (указаны также команды, которые будут выполняться на устройстве):
* all
* hardware
 * dir
 * show version
 * show memory statistics
* config
 * show version
 * show running-config
* interfaces
 * dir
 * show version
 * show interfaces
 * show ipv6 interface
 * show lldp
 * show lldp neighbors detail

Если нужно указать, что нужно собрать все факты:
```
- ios_facts:
    gather_subset: all
    provider: "{{ cli }}"
```

Собрать только подмножество interfaces:
```
- ios_facts:
    gather_subset:
      - interfaces
    provider: "{{ cli }}"
```

Собрать всё, кроме hardware:
```
- ios_facts:
    gather_subset:
      - "!hardware"
    provider: "{{ cli }}"
```

### Использование модуля

Пример playbook 5_ios_facts.yml с использованием модуля ios_facts (собираются все факты):
```
---

- name: Collect IOS facts
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: Facts
      ios_facts:
        gather_subset: all
        provider: "{{ cli }}"
```


```
$ ansible-playbook 5_ios_facts.yml
```

![5_ios_facts](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/5_ios_facts.png)



Для того, чтобы посмотреть, какие именно факты собираются с устройства, можно добавить флаг -v (информация сокращена):
```
$ ansible-playbook 5_ios_facts.yml -v
Using /home/nata/pyneng_course/chapter15/ansible.cfg as config file
```

![5_ios_facts](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/5_ios_facts_verbose.png)

### Сохранение фактов

В том виде, в котором информация отображается в режиме verbose, довольно сложно понять какая информация собирается об устройствах.
Для того, чтобы лучше понять какая информация собирается об устройствах, в каком формате, скопируем полученную информацию в файл.

Для этого мы будем использовать модуль copy.

Playbook 5a_ios_facts.yml собирает всю информацию об устройствах и записывает в разные файлы (создайте каталог all_facts перед запуском playbook):
```
---

- name: Collect IOS facts
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: Facts
      ios_facts:
        gather_subset: all
        provider: "{{ cli }}"
      register: ios_facts_result

    - name: Copy facts to files
      copy:
        content: "{{ ios_facts_result | to_nice_json }}"
        dest: "all_facts/{{inventory_hostname}}_facts.json"
```

Модуль copy позволяет копировать файлы с управляющего хоста (на котором установлен Ansible) на удаленный хост.
Но, так как в этом случае, мы указываем параметр connection: local, файлы будут скопированы на локальный хост.

Чаще всего, модуль copy используется таким образом:
```
- copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
```

Но, в нашем случае, нет исходного файла, содержимое которого нужно скопировать.
Вместо этого есть содержимое переменной ios_facts_result.
Его нам нужно перенести в файл all_facts/{{inventory_hostname}}_facts.json.

Для того чтобы перенести содержимое переменной в файл, в модуле copy, вместо src, используется параметр content.

В строке ```content: "{{ ios_facts_result | to_nice_json }}"```
* параметр to_nice_json - это фильтр Jinja2, который преобразует информацию переменной в формат, в котором удобней читать информацию
* когда нам нужно использовать переменную в формате Jinja2, она должна быть заключена в двойные фигурные скобки, а также указана в двойных кавычках

Засчет использования в пути dest имени устройства, мы получаем уникальные файлы для каждого устройства.


Результат выполнения playbook:
```
$ ansible-playbook 5a_ios_facts.yml
```

![5a_ios_facts](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/5a_ios_facts.png)

После этого, в каталоге all_facts находятся такие файлы:
```
192.168.100.1_facts.json
192.168.100.2_facts.json
192.168.100.3_facts.json
```

Содержимое файла all_facts/192.168.100.1_facts.json:
```
{
    "ansible_facts": {
        "ansible_net_all_ipv4_addresses": [
            "192.168.200.1",
            "192.168.100.1",
            "10.1.1.1"
        ],
        "ansible_net_all_ipv6_addresses": [],
        "ansible_net_config": "Building configuration...\n\nCurrent configuration :
...
```


Сохранение информации об устройствах, не только поможет разобраться, какая информация собирается, но и полезно для дальнейшего использования информации.
Например, мы можем использовать факты об устройстве в шаблоне.

Если вы будете повторно запускать playbook, и при этом, факты об устройстве не изменились, Ansible не будет изменять информацию в файлах.
Если же информация изменилась, для соответствующего устройства, будет выставлен статус changed.
Таким образом, вы всегда будете знать, что что-то изменилось.

Повторный запуск playbook (без изменений):
```
$ ansible-playbook 5a_ios_facts.yml
```

![5a_ios_facts](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/5a_ios_facts_no_change.png)

### Изменения с опцией --diff

В Ansible можно не только увидеть, что изменения произошли, но и увидеть какие именно изменения были сделаны.
Например, в ситуации с сохранением фактов об устройстве это может быть очень полезно.

Пример запуска playbook с опцией --diff и с внесенными изменениями на одном из устройств:
```
$ ansible-playbook 5a_ios_facts.yml --diff --limit=192.168.100.1
```

![5a_ios_facts](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/5a_ios_facts_diff.png)

Таким образом мы не только знаем, что были внесены изменения, но и знаем на каком устройстве и какие именно.

