## before

Параметр __before__ указывает какие действия выполнить до команд в списке lines.

Команды, которые указаны в параметре before:
* выполняются только если должны быть внесены изменения.
* при этом они будут выполнены, независимо от того есть они в конфигурации или нет.

Параметр before полезен в ситуациях, когда вам нужно выполнить какуе-то действия перед выполнением команд в списке lines.
При этом, также как и параметр after, параметр before не влияет на то, какие команды сравниваются с конфигурацией.
То есть, по-прежнему, сравниваются только команды в списке lines.

Playbook 6g_ios_config_before.yml:
```yml
---

- name: Run cfg commands on router
  hosts: 192.168.100.1
  gather_facts: false
  connection: local

  tasks:

    - name: Config ACL
      ios_config:
        before:
          - no ip access-list extended IN_to_OUT
        parents:
          - ip access-list extended IN_to_OUT
        lines:
          - permit tcp 10.0.1.0 0.0.0.255 any eq www
          - permit tcp 10.0.1.0 0.0.0.255 any eq 22
          - permit icmp any any
        provider: "{{ cli }}"
```

В playbook 6g_ios_config_before.yml мы сначала удаляем ACL IN_to_OUT с помощью параметра before, а затем создаем его заново.
Таким образом мы будем уверены всегда, что в этом ACL находятся только те строки, которые мы задали в списке lines.

Запуск playbook с изменениями:
```
$ ansible-playbook 6g_ios_config_before.yml -v
```
![6g_ios_config_before](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6g_ios_config_before.png)


Запуск playbook без изменений (команда в списке before не выполняется):
```
$ ansible-playbook 6g_ios_config_before.yml -v
```
![6g_ios_config_before_no_updates](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6g_ios_config_before_no_updates.png)

