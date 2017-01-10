## replace

Параметр replace указывает как именно  нужно заменять конфигурацию:
* __line__ - в этом режиме отправляются только те команды, которых нет в конфигурации. Этот режим используется по умолчанию
* __block__ - в этом режиме отправляются все команды, если хотя бы одной команды нет

### replace: line

Режим ```replace: line``` - это режим работы по умолчанию.
В этом режиме, если были обнаружены изменения, отправляются только недостающие строки.

Например, на маршрутизаторе такой ACL:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit icmp any any
```

Попробуем запустить такой playbook 6i_ios_config_replace_line.yml:
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
          - deny   ip any any
        provider: "{{ cli }}"
```

Выполнение playbook:
```
$ ansible-playbook 6i_ios_config_replace_line.yml -v
```
![6i_ios_config_replace_line](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6i_ios_config_replace_line.png)


После этого на маршрутизаторе такой ACL:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 deny   ip any any
```

У нас уже была такая же ситуация, когда мы рассматривали параметр match, так как именно параметр replace говорит каким образом заменять команды.

В данном случае, модуль проверил каких команд не хватает в ACL (так как режим по умолчанию match: line), обнаружил, что не хватает команды ```deny ip any any``` и добавил её.
Но, так как мы сначала удаляем ACL, а затем применяем список команд lines, получилось, что у нас теперь ACL с одной строкой.

В таких ситуациях  подходит режим ```replace: block```.

### replace: block

В режиме ```replace: block``` отправляются все команды из списка lines (и parents), если на устройстве нет хотя бы одной из этих команд.

Повторим предыдущий пример.

ACL на маршрутизаторе:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit icmp any any
```

Playbook 6i_ios_config_replace_block.yml:
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
          - deny   ip any any
        replace: block
        provider: "{{ cli }}"
```

Выполнение playbook:
```
$ ansible-playbook 6i_ios_config_replace_block.yml -v
```
![6i_ios_config_replace_block](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6i_ios_config_replace_block.png)


В результате на маршрутизаторе такой ACL:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit icmp any any
 deny   ip any any
```

