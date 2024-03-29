.. meta::
   :http-equiv=Content-Type: text/html; charset=utf-8


Переменные
==========

Переменной может быть, например: 

* информация об устройстве, которая собрана как факт, а затем используется в шаблоне. 
* в переменные можно записывать полученный вывод команды. 
* переменная может быть указана вручную в playbook

Имена переменных
----------------

В Ansible есть определенные ограничения по формату имен переменных: 

* Переменные могут состоять из букв, чисел и символа ``_`` 
* Переменные должны начинаться с буквы

Кроме того, можно создавать словари с переменными (в формате YAML):

::

    R1:
      IP: 10.1.1.1/24
      DG: 10.1.1.100

Обращаться к переменным в словаре можно двумя вариантами:

::

    R1['IP']
    R1.IP

Правда, при использовании второго варианта могут быть проблемы, если
название ключа совпадает с зарезервированным словом (методом или
атрибутом) в Python или Ansible.

Где можно определять переменные
-------------------------------

Переменные можно создавать: 

* в инвентарном файле 
* в playbook 
* в специальных файлах для группы/устройства 
* в отдельных файлах, которые добавляются в playbook через include (как в Jinja2) 
* в ролях, которыезатем используются 
* можно даже передавать переменные при вызове playbook

Также можно использовать факты, которые были собраны про устройство, как
переменные.

Переменные в инвентарном файле
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В инвентарном файле можно указывать переменные для группы:

::

    [cisco_routers]
    192.168.100.1
    192.168.100.2
    192.168.100.3

    [cisco_switches]
    192.168.100.100

    [cisco_routers:vars]
    ntp_server=192.168.255.100
    log_server=10.255.100.1

Переменные ntp_server и log_server относятся к группе cisco_routers и
могут использоваться, например, при генерации конфигурации на основе
шаблона.

Переменные в playbook
~~~~~~~~~~~~~~~~~~~~~

Переменные можно задавать прямо в playbook. Это может быть удобно тем,
что переменные находятся там же, где все действия.

Например, можно задать переменную interfaces в playbook
таким образом:

::

    ---

    - name: Run show commands on routers
      hosts: cisco_routers
      gather_facts: false


      vars:
        interfaces: sh ip int br

      tasks:

        - name: run sh ip int br
          ios_command:
            commands: "{{interfaces}}"

        - name: run sh ip arp
          ios_command:
            commands: sh ip arp


Переменные в специальных файлах для группы/устройства
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ansible позволяет хранить переменные для группы/устройства в специальных
файлах: 

* Для групп устройств, переменные должны находиться в каталоге
  group_vars, в файлах, которые называются, как имя группы. 
  Кроме того, можно создавать в каталоге group_vars файл all, в котором будут
  находиться переменные, которые относятся ко всем группам. 
* Для конкретных устройств, переменные должны находиться в каталоге
  host_vars, в файлах, которые соответствуют имени или адресу хоста. 
 *Все файлы с переменными должны быть в формате YAML. Расширение файла
 может быть таким: yml, yaml, json или без расширения 
* каталоги group_vars и host_vars должны находиться в том же каталоге, что и
  playbook, или могут находиться внутри каталога inventory (первый вариант
  более распространенный). Если каталоги и файлы названы правильно и
  расположены в указанных каталогах, Ansible сам распознает файлы и будет 
  использовать переменные.

Например, если инвентарный файл myhosts.ini выглядит так:

::

    [cisco_routers]
    192.168.100.1
    192.168.100.2
    192.168.100.3

    [cisco_switches]
    192.168.100.100

Можно создать такую структуру каталогов:

::

    ├── group_vars                 _
    │   ├── all.yml                 |
    │   ├── cisco_routers.yml       |  Каталог с переменными для групп устройств
    │   └── cisco_switches.yml     _|
    |
    ├── host_vars                  _
    │   ├── 192.168.100.1           |
    │   ├── 192.168.100.2           |
    │   ├── 192.168.100.3           |  Каталог с переменными для устройств 
    │   └── 192.168.100.100        _|
    |
    └── myhosts.ini                 |  Инвентарный файл

Ниже пример содержимого файлов переменных для групп устройств и для
отдельных хостов.

group_vars/all.yml (в этом файле указываются значения по умолчанию,
которые относятся ко всем устройствам):

::

    ---

    ansible_connection: network_cli
    ansible_network_os: ios
    ansible_user: cisco
    ansible_password: cisco
    ansible_become: yes
    ansible_become_method: enable
    ansible_become_pass: cisco


В данном случае указываются переменные, которые предопределены самим
Ansible.

group_vars/cisco_routers.yml

::

    ---

    log_server: 10.255.100.1
    ntp_server: 10.255.100.1
    users:
      user1: pass1
      user2: pass2
      user3: pass3

В файле group_vars/cisco_routers.yml находятся переменные, которые
указывают IP-адреса Log и NTP серверов и нескольких пользователей. Эти
переменные могут использоваться, например, в шаблонах конфигурации.

group_vars/cisco_switches.yml

::

    ---

    vlans:
      - 10
      - 20
      - 30

В файле group_vars/cisco_switches.yml указана переменная vlans со
списком VLANов.

Файлы с переменными для хостов однотипны, и в них меняются только адреса
и имена:

Файл host_vars/192.168.100.1.yml

::

    ---

    hostname: london_r1
    mgmnt_loopback: 100
    mgmnt_ip: 10.0.0.1
    ospf_ints:
      - 192.168.100.1
      - 10.0.0.1
      - 10.255.1.1

Приоритет переменных
--------------------

.. note::
    В этом разделе не рассматривается размещение переменных: 

        * в отдельных файлах, которые добавляются в playbook через include (как в Jinja2) 
        * в ролях, которые затем используются 
        * передача переменных при вызове playbook

Чаще всего, переменная с определенным именем только одна, но иногда
может понадобиться создать переменную в разных местах, и тогда нужно
понимать, в каком порядке Ansible перезаписывает переменные.

Приоритет переменных (последние значения переписывают предыдущие): 

* переменные в инвентарном файле 
* переменные для группы хостов в инвентарном файле 
* переменные для хостов в инвентарном файле 
* переменные в каталоге group_vars 
* переменные в каталоге host_vars 
* факты хоста 
* переменные сценария (play) 
* переменные, полученные через параметр register 
* переменные, которые передаются при вызове playbook через параметр --extra-vars
  (всегда наиболее приоритетные)

`Более полный список в документации <https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#sts=Variable%20precedence:%20Where%20should%20I%20put%20a%20variable?%C2%B6>`__
