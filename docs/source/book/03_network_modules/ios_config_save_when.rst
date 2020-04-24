save_when
----------

Параметр **save_when** позволяет указать, нужно ли сохранять текущую
конфигурацию в стартовую.

Доступные варианты значений: 

* always - всегда сохранять конфигурацию (в этом случае флаг modified будет равен True) 
* never (по умолчанию) - не сохранять конфигурацию 
* modified - в этом случае конфигурация сохраняется только при наличии изменений


Playbook 4_ios_config_save_when.yml:

::

    ---

    - name: Run cfg commands on routers
      hosts: cisco-routers

      tasks:

        - name: Config line vty
          ios_config:
            parents:
              - line vty 0 4
            lines:
              - login local
              - transport input ssh telnet
            save_when: modified


Выполнение playbook:

::

    $ ansible-playbook 4_ios_config_save_when.yml

.. figure:: https://raw.githubusercontent.com/natenka/PyNEng/master/images/15_ansible/6c_ios_config_save_2.png
