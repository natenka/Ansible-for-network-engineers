Include
-------

До сих пор, каждый playbook был отдельным файлом. И, хотя для простых
сценариев, такой вариант подходит, когда задач становится больше, может
понадобиться выполнять одни и те же действия в разных playbook. И было
бы намного удобней, если бы можно было разбить playbook на блоки,
которые можно повторно использовать (как в случае с функциями).

Это можно сделать с помощью выражений include (и с помощью ролей,
которые мы будем рассматриваться в следующем разделе).

С помощью выражения include, в playbook можно добавлять: \* задачи \*
handlers \* сценарий (play) \* playbook \* файлы с переменными
(используют другое ключевое слово)

Task include
~~~~~~~~~~~~

Task include позволяют подключать в текущий playbook файлы с задачами.

Например, создадим каталог tasks и добавим в него два файла с задачами.

Файл tasks/cisco\_vty\_cfg.yml:

.. code:: yml

    ---

    - name: Config line vty
      ios_config:
        parents:
          - line vty 0 4
        lines:
          - exec-timeout 30 0
          - login local
          - history size 100
          - transport input ssh
        provider: "{{ cli }}"
      notify: save config

Файл tasks/cisco\_ospf\_cfg.yml:

.. code:: yml

    ---

    - name: Config ospf
      ios_config:
        src: templates/ospf.j2
        provider: "{{ cli }}"
      notify: save config

Шаблон templates/ospf.j2 (переменные, которые используются в шаблоне,
находятся в файлах с переменными для каждого устройства, в каталоге
host\_vars):

::

    router ospf 1
     router-id {{ mgmnt_ip }}
     ispf
     auto-cost reference-bandwidth 10000
    {% for ip in ospf_ints %}
     network {{ ip }} 0.0.0.0 area 0
    {% endfor %}

Теперь создадим playbook, который будет использовать созданные файлы с
задачами.

Playbook 8\_playbook\_include\_tasks.yml:

.. code:: yml

    ---

    - name: Run cfg commands on routers
      hosts: cisco-routers
      gather_facts: false
      connection: local

      tasks:

        - name: Disable services
          ios_config:
            lines:
              - no ip http server
              - no ip http secure-server
              - no ip domain lookup
            provider: "{{ cli }}"
          notify: save config

        - include: tasks/cisco_ospf_cfg.yml
        - include: tasks/cisco_vty_cfg.yml

      handlers:

        - name: save config
          ios_command:
            commands:
              - write
            provider: "{{ cli }}"

В этом playbook специально создана обычная задача. А также handler,
который мы использовали в предыдущем разделе. Он вызывается и из задачи,
которая находится в playbook, и из задач в подключаемых файлах.

Обратите внимание, что строки include находятся на том же уровне, что и
задача.

    В конфигурации R1 внесены изменения, чтобы playbook мог выполнить
    конфигурацию устройства.

Запуск playbook с изменениями:

::

    $ ansible-playbook 8_playbook_include_tasks.yml

.. figure:: https://raw.githubusercontent.com/natenka/Ansible-for-network-engineers/master/images/8_playbook_include_tasks.png
   :alt: 8\_playbook\_include\_tasks

   8\_playbook\_include\_tasks
При выполнении playbook, задачи которые мы добавили через include
работают так же, как если бы они находились в самом playbook.

Таким образом мы можем делать отдельные файлы с задачами, которые
настраивают определенную функциональность, а затем собирать их в нужной
комбинации в итоговом playbook.

Передача переменных в include
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

При использовании include, задачам можно передавать аргументы.

Например, когда мы использовали команду ntc\_show\_command из модуля
ntc-ansible, нужно было задать ряд параметров. Так как они не вынесены в
отдельную переменную, как в случае с модулями ios\_config, ios\_command
и ios\_facts, довольно не удобно каждый раз их описывать.

Попробуем вынести задачу с использованием ntc\_show\_command в отдельный
файл tasks/ntc\_show.yml:

.. code:: yml

    ---

    - ntc_show_command:
        connection: ssh
        platform: "cisco_ios"
        command: "{{ ntc_command }}"
        host: "{{ inventory_hostname }}"
        username: "cisco"
        password: "cisco"
        template_dir: "library/ntc-ansible/ntc-templates/templates"

В этом файле указаны две переменные: ntc\_command и inventory\_hostname.
С переменной inventory\_hostname мы уже сталкивались раньше, она
автоматически становится равной текущеву устройству, для которого
Ansible выполняет задачу.

А значение переменной ntc\_command мы будем передавать из playbook.

Playbook 8\_playbook\_include\_tasks\_var.yml:

.. code:: yml

    ---

    - name: Run cfg commands on routers
      hosts: 192.168.100.1
      gather_facts: false
      connection: local

      tasks:

        - include: tasks/cisco_ospf_cfg.yml
        - include: tasks/ntc_show.yml ntc_command="sh ip route"

      handlers:

        - name: save config
          ios_command:
            commands:
              - write
            provider: "{{ cli }}"

В таком варианте, нам достаточно указать какую команду передать
ntc\_show\_command.

Переменные можно передавать и таким образом:

.. code:: yml

      tasks:

        - include: tasks/cisco_ospf_cfg.yml
        - include: tasks/ntc_show.yml
          vars:
            ntc_command: "sh ip route"

Такой вариант удобнее, когда вам нужно передать несколько переменных.

Handler include
~~~~~~~~~~~~~~~

Include можно использовать и в разделе handlers.

Например, перенесем handler из предыдущих примеров в отдельный файл
handlers/cisco\_save\_cfg.yml:

.. code:: yml

    ---

    - name: save config
      ios_command:
        commands:
          - write
        provider: "{{ cli }}"

И добавим его в playbook 8\_playbook\_include\_handlers.yml через
include:

.. code:: yml

    ---

    - name: Run cfg commands on routers
      hosts: cisco-routers
      gather_facts: false
      connection: local

      tasks:

        - name: Disable services
          ios_config:
            lines:
              - no ip http server
              - no ip http secure-server
              - no ip domain lookup
            provider: "{{ cli }}"
          notify: save config

        - include: tasks/cisco_ospf_cfg.yml
        - include: tasks/cisco_vty_cfg.yml

      handlers:

        - include: handlers/cisco_save_cfg.yml

Запуск playbook:

::

    $ ansible-playbook 8_playbook_include_handlers.yml -v

.. figure:: https://raw.githubusercontent.com/natenka/Ansible-for-network-engineers/master/images/8_playbook_include_handlers.png
   :alt: 8\_playbook\_include\_handlers

   8\_playbook\_include\_handlers
Playbook выполняет handler, как-будто он находится в playbook. Таким
образом можно легко добавлять handler в любой playbook.

Play/playbook include
~~~~~~~~~~~~~~~~~~~~~

С помощью выражения include можно добавить в playbook и целый сценарий
(play) или другой playbook. От добавления задач это будет отличаться
только уровнем, на котором выполняется include.

Например, у нас есть такой сценарий 8\_play\_to\_include.yml:

.. code:: yml

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

Добавим его в playbook 8\_playbook\_include\_play.yml:

.. code:: yml

    ---

    - name: Run cfg commands on routers
      hosts: cisco-routers
      gather_facts: false
      connection: local

      tasks:

        - name: Disable services
          ios_config:
            lines:
              - no ip http server
              - no ip http secure-server
              - no ip domain lookup
            provider: "{{ cli }}"
          notify: save config

        - include: tasks/cisco_ospf_cfg.yml
        - include: tasks/cisco_vty_cfg.yml

      handlers:

        - include: handlers/cisco_save_cfg.yml

    - include: 8_play_to_include.yml

Если выполнить playbook, то все задачи из файла 8\_play\_to\_include.yml
выполняются точно так же, как и те, которые находятся в playbook (вывод
сокращен):

::

    $ ansible-playbook 8_playbook_include_play.yml

.. figure:: https://raw.githubusercontent.com/natenka/Ansible-for-network-engineers/master/images/8_playbook_include_play.png
   :alt: 8\_playbook\_include\_play

   8\_playbook\_include\_play
Vars include
~~~~~~~~~~~~

Несмотря на то, что файлы с переменными могут быть вынесены в каталоги
host\_vars и group\_vars, и разбиты на части, которые относятся ко всем
устройствам, к группе или к конкретному устройству, иногда не хватает
этой иерархии и файлы с переменными становятся слишком большими. Но и
тут Ansible поддерживает возможность создавать дополнительную иерархию.

Можно создавать отдельные файлы с переменными, которые будут относиться,
например, к настройке определенного функционала.

include\_vars
^^^^^^^^^^^^^

Например, создадим каталог vars и добавим в него файл
vars/cisco\_bgp\_general.yml

.. code:: yml

    ---

    as: 65000
    network: 120.0.0.0 mask 255.255.252.0
    ttl_security_hops: 3
    send_community: true
    update_source_int: Loopback0
    ibgp_neighbors:
      - 10.0.0.1
      - 10.0.0.2
      - 10.0.0.3
      - 10.0.0.4
    ebgp_neighbors:
      - ip: 15.0.0.5
        as: 500
      - ip: 26.0.0.6
        as: 600

Переменные будем использовать для генерации конфигурации BGP по шаблону
templates/bgp.j2:

::

    router bgp {{ as }}
     network {{ network }}
     {% for n in ibgp_neighbors %}
     neighbor {{ n }} remote-as {{ as }}
     neighbor {{ n }} update-source {{ update_source_int }}
     {% endfor %}
     {% for extn in ebgp_neighbors %}
     neighbor {{ extn.ip }} remote-as {{ extn.as }}
     neighbor {{ extn.ip }} ttl-security hops {{ ttl_security_hops }}
     {% if send_community == true %}
     neighbor {{ extn.ip }} send-community
     {% endif %}
     {% endfor %}

    Шаблон подразумевает настройку одного маршрутизатора, просто чтобы
    показать как добавлять переменные из файла.

Итоговый playbook 8\_playbook\_include\_vars.yml

.. code:: yml

    ---

    - name: Run cfg commands on router
      hosts: 192.168.100.1
      gather_facts: false
      connection: local

      tasks:

        - name: Include BGP vars
          include_vars: vars/cisco_bgp_general.yml

        - name: Config BGP
          ios_config:
            src: templates/bgp.j2
            provider: "{{ cli }}"

        - name: Show BGP config
          ios_command:
            commands: sh run | s ^router bgp
            provider: "{{ cli }}"
          register: bgp_cfg

        - name: Debug registered var
          debug: var=bgp_cfg.stdout_lines

Обратите внимание, что переменные из файла подключаются отдельной
задачей (в данном случае, можно было бы обойтись без имени задачи):

.. code:: yml

        - name: Include BGP vars
          include_vars: vars/cisco_bgp_general.yml

Выполнение playbook выглядит так:

::

    $ ansible-playbook 8_playbook_include_vars.yml

.. figure:: https://raw.githubusercontent.com/natenka/Ansible-for-network-engineers/master/images/8_playbook_include_vars.png
   :alt: 8\_playbook\_include\_vars

   8\_playbook\_include\_vars
    Модуль include\_vars поддерживает большое количество вариантов
    использования. Подробнее об этом можно почитать в `документации
    модуля <http://docs.ansible.com/ansible/include_vars_module.html>`__.

vars\_files
^^^^^^^^^^^

Второй вариант добавления файлов с переменными - использование
vars\_files.

Его отличие в том, что мы создаем переменные на уровне сценария (play),
а не на уровне задаче.

Пример playbook 8\_playbook\_include\_vars\_files.yml:

.. code:: yml

    ---

    - name: Run cfg commands on router
      hosts: 192.168.100.1
      gather_facts: false
      connection: local

      vars_files:
        - vars/cisco_bgp_general.yml

      tasks:

        - name: Config BGP
          ios_config:
            src: templates/bgp.j2
            provider: "{{ cli }}"

        - name: Show BGP config
          ios_command:
            commands: sh run | s ^router bgp
            provider: "{{ cli }}"
          register: bgp_cfg

        - name: Debug registered var
          debug: var=bgp_cfg.stdout_lines

Результат выполнения будет в целом аналогичен предыдущему выводу, но,
так как файл с переменными указывался через vars\_files, загрузка
переменных не будет видна как отдельная задача:

::

    $ ansible-playbook 8_playbook_include_vars_files.yml

.. figure:: https://raw.githubusercontent.com/natenka/Ansible-for-network-engineers/master/images/8_playbook_include_vars_files.png
   :alt: 8\_playbook\_include\_vars\_files

   8\_playbook\_include\_vars\_files

