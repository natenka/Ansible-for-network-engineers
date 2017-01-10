## Работа с результатами выполнения модуля

До сих пор мы только отправляли команды на устройства и никак не работали с результатом.
Как минимум, нам нужно знать как посмотреть на вывод команды.

В этом разделе мы посмотрим на то, как можно получать результаты выполнения команды, отображать их, использовать в следующих действиях.

Примеры будут использовать модуль raw.
Но аналогичные принципы будут работать и с другими модулями.

### verbose

Мы уже видели один из способов показать результат выполнения модуля - флаг verbose.

Конечно, вывод не особо удобно читать, но, как минимум, мы видим, что команды выполнились.
Также этот флаг позволяет подробно посмотреть какие шаги выполняет Ansible.

Пример запуска playbook с флагом verbose (вывод сокращен):
```
ansible-playbook 1_show_commands_with_raw.yml -v
```

![Verbose playbook](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/playbook-verbose.png)

При увеличении количества букв v в флаге, вывод становится более подробным.
Попробуйте вызывать этот же playbook и добавлять к флагу буквы v (5 и больше показывают одинаковый вывод).

Несмотря на то, что вывод не очень приятен для восприятия, мы видим, что в результаты выполнения задачи, мы получаем объект в формате JSON, с такими полями:
* changed - ключ, который указывает были ли внесены изменения
* rc - return code. Это поле будет появляться в выводе тех модулей, которые выполняют какие-то команды
* stderr - ошибки, при выполнении команды. Это поле будет появляться в выводе тех модулей, которые выполняют какие-то команды
* stdout - вывод команды
* stdout_lines - вывод команды разбитый построчно


### register

Параметр __register__ позволяет сохранить результат выполнения модуля в переменную.
Затем эта переменная может использоваться в шаблонах, в принятии решений о ходе сценария и отображении вывода.

Попробуем сохранить результат выполнения команды.
Для этого будем использовать такой playbook:
```
---

- name: Run show commands on routers
  hosts: cisco-routers
  gather_facts: false

  tasks:

    - name: run sh ip int br
      raw: sh ip int br | ex unass
      register: sh_ip_int_br_result
```

Если запустить этот playbook, вывод не будет отличаться, так как мы отлько записали вывод в переменную, но ничего с ней не делаем.
Попробуем отобразить результат выполнения команды - для этого будем использовать модуль debug.


### debug

Модуль debug позволяет отображать информацию на стандартный поток вывода.
Это может произвольная строка, переменная, которую мы сохранили ранее или какие-то переменные, которые определили мы или те, которые получены в результате сбора фактов об устройстве.


Для отображения сохраненный результатов выполнения команды, добавим задание в playbook:
```
---

- name: Run show commands on routers
  hosts: cisco-routers
  gather_facts: false

  tasks:

    - name: run sh ip int br
      raw: sh ip int br | ex unass
      register: sh_ip_int_br_result

    - name: Debug registered var
      debug: var=sh_ip_int_br_result.stdout_lines
```

Обратите внимание, что мы выводим не всё содержимое переменной sh_ip_int_br_result, а только содержимое stdout_lines.
Таким образом вы увидим структурированный вывод.

Результат запуска playbook будет выглядеть  так:
```
$ ansible-playbook 2_register_vars.yml
```

![Verbose playbook](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/2_register_vars.png)


### register, debug, when

С помощью ключевого слова __when__, можно указать условие, при выполнении которого, задача выполняется.
Если условие не выполняется, то задача пропускается.

Например, создадим такой playbook 3_register_debug_when.yml:
```
---

- name: Run show commands on routers
  hosts: cisco-routers
  gather_facts: false

  tasks:

    - name: run sh ip int br
      raw: sh ip int bri | ex unass
      register: sh_ip_int_br_result

    - name: Debug registered var
      debug:
        msg: "Error in command"
      when: "'invalid' in sh_ip_int_br_result.stdout"
```

В последнем задании у нас несколько изменений:
* модуль debug теперь отображает не содержимое сохраненной переменной, а сообщение, которое мы указали в переменной msg.
* условие when позволяет указать, что данное задание будет выполняться только если условие будет выполнено
 * when: "'invalid' in sh_ip_int_br_result.stdout" - это условие означает, что задача будет выполнена только в том случае, если в выводе sh_ip_int_br_result.stdout будет найдена строка invalid (например, когда неправильно введена команда)

Сначала попробуем выполнить playbook:
```
$ ansible-playbook 3_register_debug_when.yml
```

![Verbose playbook](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/3_register_debug_when_skip.png)

Обратите внимание на сообщения skipping - это означает, что задача не выполнялась для указанных устройств.
Не выполнилась она потому, что условие в when не было выполнено.

Теперь попробуем тот же playbook, но сделаем ошибку в команде:
```
---

- name: Run show commands on routers
  hosts: cisco-routers
  gather_facts: false

  tasks:

    - name: run sh ip int br
      raw: shh ip int bri | ex unass
      register: sh_ip_int_br_result

    - name: Debug registered var
      debug:
        msg: "Error in command"
      when: "'invalid' in sh_ip_int_br_result.stdout"
```

Теперь результат выполнения будет таким:
```
$ ansible-playbook 3_register_debug_when.yml
```

![Verbose playbook](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/3_register_debug_when.png)

Теперь мы видим сообщение, которое было указано в задаче для модуля debug, так как команда была с ошибкой.

С помощью условий в when, можно не только генерировать какие-то сообщения с модулем debug, но и контролировать то, какие действия будут выполняться, в зависимости от условия.
