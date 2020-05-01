Handlers
--------

Handlers - это специальные задачи. Они вызываются из других задач
ключевым словом **notify**.

Эти задачи срабатывают после выполнения всех задач в сценарии (play).
При этом, если несколько задач вызвали одну и ту же задачу через notify,
она выполниться только один раз.

Handlers описываются в своем подразделе playbook - handlers, так же, как
и задачи. Для них используется такой же синтаксис, как и для задач.

Пример использования handlers (playbook 8_handlers.yml):

::

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
            provider: "{{ cli }}"
          notify: save config

        - name: Send config commands
          ios_config:
            lines:
              - service password-encryption
              - no ip http server
              - no ip http secure-server
              - no ip domain lookup
            provider: "{{ cli }}"
          notify: save config

      handlers:

        - name: save config
          ios_command:
            commands:
              - write
            provider: "{{ cli }}"

Запуск playbook с изменениями:

::

    $ ansible-playbook 8_handlers.yml

.. figure:: https://raw.githubusercontent.com/natenka/Ansible-for-network-engineers/master/images/8_handler.png

Обратите внимание, что handler выполняется только один раз.

Запуск того же playbook с изменениями и режимом verbose:

::

    $ ansible-playbook 8_handlers.yml -v

.. figure:: https://raw.githubusercontent.com/natenka/Ansible-for-network-engineers/master/images/8_handlers_verbose.png

Запуск playbook без изменений:

::

    $ ansible-playbook 8_handlers.yml

.. figure:: https://raw.githubusercontent.com/natenka/Ansible-for-network-engineers/master/images/8_handlers_no_change.png


Так как в задачах не нужно выносить изменений, handler также не
выполняется.
