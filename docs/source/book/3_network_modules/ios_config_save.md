## save

Параметр __save__ позволяет указать нужно ли сохранять текущую конфигурацию в стартовую. По умолчанию, значение параметра - __no__.

Доступные варианты значений:
* no (или false)
* yes (или true)

К сожалению, на данный момент (версия ansible 2.2), этот параметр не отрабатывает корректно, так как на устройство отправляется команда copy running-config startup-config, но, при этом, не отправляется подтверждение на сохранение.
Из-за этого, при запуске playbook с параметром save выставленным в yes, появляется такая ошибка:
```
$ ansible-playbook 4_ios_config_save.yml
```

![6c_ios_config_save]({{ book.ansible_img_path }}6c_ios_config_save.png)

Но, можно самостоятельно сделать сохранение, используя модуль ios_command.

{% raw %}
Playbook 4_ios_config_save.yml:
```yml
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
        #save: yes - в версии 2.2 не работает корректно
        provider: "{{ cli }}"
      register: cfg

    - name: Save config
      ios_command:
        commands:
          - write
        provider: "{{ cli }}"
      when: cfg.changed
```
{% endraw %}

> Надо внести изменения на маршрутизаторе 192.168.100.1. Например, изменить строку transport input all на transport input ssh.

Выполнение playbook:
```
$ ansible-playbook 4_ios_config_save.yml
```

![6c_ios_config_save]({{ book.ansible_img_path }}6c_ios_config_save_2.png)

