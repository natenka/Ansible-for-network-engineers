.. meta::
   :http-equiv=Content-Type: text/html; charset=utf-8


Конфигурационный файл
---------------------

Настройки Ansible можно менять в конфигурационном файле.

Конфигурационный файл Ansible может храниться в разных местах (файлы
перечислены в порядке уменьшения приоритета): 

* ANSIBLE_CONFIG (переменная окружения) 
* ansible.cfg (в текущем каталоге) 
* ~/.ansible.cfg (в домашнем каталоге пользователя) 
* /etc/ansible/ansible.cfg

Ansible ищет файл конфигурации в указанном порядке и использует первый
найденный (конфигурация из разных файлов не совмещается).

В конфигурационном файле можно менять множество параметров. Полный
список параметров и их описание можно найти в
`документации <https://docs.ansible.com/ansible/latest/reference_appendices/config.html#common-options>`__.

В текущем каталоге должен быть инвентарный файл myhosts.ini:

::

    [cisco_routers]
    192.168.100.1
    192.168.100.2
    192.168.100.3


В текущем каталоге надо создать такой конфигурационный файл ansible.cfg:

::

    [defaults]

    inventory = ./myhosts.ini
    remote_user = cisco
    ask_pass = True

Настройки в конфигурационном файле: 

* ``[defaults]`` - эта секция конфигурации описывает общие параметры по умолчанию 
* ``inventory = ./myhosts`` - параметр inventory позволяет указать
  местоположение инвентарного файла. Если настроить этот параметр, не придется указывать, 
  где находится файл, при каждом запуске Ansible 
* ``remote_user = cisco`` - от имени какого пользователя будет подключаться Ansible 
* ``ask_pass = True`` - этот параметр аналогичен опции --ask-pass в 
  командной строке. Если он выставлен в конфигурации
  Ansible, то уже не нужно указывать его в командной строке.

Теперь вызов ad-hoc команды будет выглядеть так:

::

    $ ansible 192.168.100.1 -m ios_command -a "commands='sh ip int br'"


gathering
~~~~~~~~~

По умолчанию Ansible собирает факты об устройствах.

Факты - это информация о хостах, к которым подключается Ansible. Эти
факты можно использовать в playbook и шаблонах как переменные.

Сбором фактов, по умолчанию, занимается модуль
`setup <https://docs.ansible.com/ansible/latest/modules/setup_module.html>`__.

Но для сетевого оборудования модуль setup не подходит, поэтому сбор
фактов надо отключить. Это можно сделать в конфигурационном файле
Ansible или в playbook.

.. note::

    Для сетевого оборудования нужно использовать отдельные модули для
    сбора фактов (если они есть). Это рассматривается в разделе
    ios_facts.

Отключение сбора фактов в конфигурационном файле:

::

    gathering = explicit

host_key_checking
~~~~~~~~~~~~~~~~~~~

Параметр host_key_checking отвечает за проверку ключей при подключении
по SSH. Если указать в конфигурационном файле
``host_key_checking=False``, проверка будет отключена.

Это полезно, когда с управляющего хоста Ansible надо подключиться к
большому количеству устройств первый раз.

Другие параметры конфигурационного файла можно посмотреть в
документации. Пример конфигурационного файла в `репозитории
Ansible <https://github.com/ansible/ansible/blob/stable-2.9/examples/ansible.cfg>`__.
