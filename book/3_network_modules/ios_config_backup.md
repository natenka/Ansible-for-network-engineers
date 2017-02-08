{% raw %}
## backup

Параметр __backup__ указывает нужно ли делать резервную копию текущей конфигурации устройства перед внесением изменений.
Файл будет копироваться в каталог backup, относительно каталога в котором находится playbook (если каталог не существует, он будет создан).

Playbook 5_ios_config_backup.yml:
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
        backup: yes
        provider: "{{ cli }}"
```

{% endraw %}

Теперь, каждый раз, когда выполняется playbook (даже если не нужно вносить изменения в конфигурацию), в каталог backup будет копироваться текущая конфигурация:
```
$ ansible-playbook 5_ios_config_backup.yml -v
```
![6d_ios_config_backup]({{ book.ansible_img_path }}6d_ios_config_backup.png)


В каталоге backup теперь находятся файлы такого вида (при каждом запуске playbook они перезаписываются):
```
192.168.100.1_config.2016-12-10@10:42:34
192.168.100.2_config.2016-12-10@10:42:34
192.168.100.3_config.2016-12-10@10:42:34
```

