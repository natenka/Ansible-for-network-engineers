{% raw %}
## match

Параметр __match__  указывает как именно нужно сравнивать команды (что считается изменением):
* __line__ - команды проверяются построчно. Этот режим используется по умолчанию
* __strict__ - должны совпасть не только сами команды, но их положение относительно друг друга
* __exact__ - команды должны в точности сопадать с конфигурацией и не должно быть никаких лишних строк
* __none__ - модуль не будет сравнивать команды с текущей конфигурацией

### match: line

Режим ```match: line``` используется по умолчанию.

В этом режиме, модуль проверяет только наличие строк, перечисленных в списке lines в соответствующем режиме.
При этом, не проверяется порядок строк.

На маршрутизаторе 192.168.100.1 настроен такой ACL:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
```

Пример использования playbook 9_ios_config_match_line.yml в режиме line:
```yml
---

- name: Run cfg commands on router
  hosts: 192.168.100.1
  gather_facts: false
  connection: local

  tasks:

    - name: Config ACL
      ios_config:
        parents:
          - ip access-list extended IN_to_OUT
        lines:
          - permit tcp 10.0.1.0 0.0.0.255 any eq www
          - permit tcp 10.0.1.0 0.0.0.255 any eq 22
          - permit icmp any any
        provider: "{{ cli }}"
```
{% endraw %}

Результат выполнения playbook:
```
$ ansible-playbook 9_ios_config_match_line.yml -v
```
![6h_ios_config_match_line]({{ book.ansible_img_path }}6h_ios_config_match_line.png)


Обратите внимание, что в списке updates только две из трёх строк ACL.
Так как в режиме lines модуль сравнивает команды независимо друг от друга, он обнаружил, что не хватает только двух команд из трех.

В итоге конфигурация на маршрутизаторе выглядит так:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 permit icmp any any
```

То есть, порядок команд поменялся.
И, хотя в этом случае, это не важно, иногда это может привести совсем не к тем результатам, которые ожидались.

Если повторно запустить playbook, при такой конфигурации, он не будет выполнять изменения, так как все строки были найдены.

### match: exact
Пример, в котором порядок команд важен.

ACL на маршрутизаторе:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 deny   ip any any
```

{% raw %}
Playbook 9_ios_config_match_exact.yml (будет постепенно дополняться):
```yml
---

- name: Run cfg commands on router
  hosts: 192.168.100.1
  gather_facts: false
  connection: local

  tasks:

    - name: Config ACL
      ios_config:
        parents:
          - ip access-list extended IN_to_OUT
        lines:
          - permit tcp 10.0.1.0 0.0.0.255 any eq www
          - permit tcp 10.0.1.0 0.0.0.255 any eq 22
          - permit icmp any any
          - deny   ip any any
        provider: "{{ cli }}"
```

{% endraw %}
Если запустить playbook, результат будет таким:
```
$ ansible-playbook 9_ios_config_match_exact.yml -v
```
![6h_ios_config_match_exact]({{ book.ansible_img_path }}6h_ios_config_match_exact_1.png)


Теперь ACL выглядит так:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 deny   ip any any
 permit icmp any any
```

Конечно же, в таком случае, последнее правило никогда не сработает.


{% raw %}
Можно добавить к этому playbook параметр before и сначала удалить ACL, а затем применять команды:
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

{% endraw %}
Если применить playbook к последнему состоянию маршрутизатора, то изменений не будет никаких, так как все строки уже есть.

Попробуем начать с такого состояния ACL:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 deny   ip any any
```

Результат будет таким:
```
$ ansible-playbook 9_ios_config_match_exact.yml -v
```
![6h_ios_config_match_exact]({{ book.ansible_img_path }}6h_ios_config_match_exact_2.png)


И, соответственно, на маршрутизаторе:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit icmp any any
```

Теперь в ACL осталась только одна строка:
* Модуль проверил каких команд не хватает в ACL (так как режим по умолчанию match: line),
* обнаружил, что не хватает команды ```permit icmp any any``` и добавил её

Но, так как в playbook ACL сначала удаляется, а затем применяется список команд lines, получилось, что в итоге в ACL одна строка.

{% raw %}
Поможет, в такой ситуации, вариант ```match: exact```:
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
        match: exact
        provider: "{{ cli }}"
```

{% endraw %}
Применение playbook 9_ios_config_match_exact.yml к текущему состоянию маршрутизатора (в ACL одна строка):
```
$ ansible-playbook 9_ios_config_match_exact.yml -v
```
![6h_ios_config_match_exact]({{ book.ansible_img_path }}6h_ios_config_match_exact_final.png)


Теперь результат такой:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit icmp any any
 deny   ip any any
```

То есть, теперь ACL выглядит точно так же, как и строки в списке lines и в том же порядке.

>**Note** В версии Ansible 2.1 match: exact работал по-другому и такой результат достигался комбинацией параметров match: exact и replace: block.
> В версии 2.2 достаточно match: exact.

И, для того чтобы окончательно разобраться с параметром ```match: exact```, ещё один пример.
{% raw %}

Закомментируем в playbook строки с удалением ACL:
```yml
---

- name: Run cfg commands on router
  hosts: 192.168.100.1
  gather_facts: false
  connection: local

  tasks:

    - name: Config ACL
      ios_config:
        #before:
        #  - no ip access-list extended IN_to_OUT
        parents:
          - ip access-list extended IN_to_OUT
        lines:
          - permit tcp 10.0.1.0 0.0.0.255 any eq www
          - permit tcp 10.0.1.0 0.0.0.255 any eq 22
          - permit icmp any any
          - deny   ip any any
        match: exact
        provider: "{{ cli }}"
```

{% endraw %}
В начало ACL добавлена строка:
```
ip access-list extended IN_to_OUT
 permit udp any any
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit icmp any any
 deny   ip any any
```

То есть, последние 4 строки выглядят так, как нужно, и в том порядке, котором нужно.
Но, при этом, есть лишняя строка.
Для варианта match: exact - это уже несовпадение.

В таком варианте, playbook будет выполняться каждый раз и пытаться применить все команды из списка lines, что не будет влиять на содержимое ACL:
```
$ ansible-playbook 9_ios_config_match_exact.yml -v
```
![6h_ios_config_match_exact]({{ book.ansible_img_path }}6h_ios_config_match_exact_final_2.png)

Это значит, что при использовании ```match:exact```, важно, чтобы был какой-то способ удалить конфигурацию, если она не соответствует тому, что должно быть (или чтобы команды перезаписывались).
Иначе, эта задача будет выполняться каждый раз, при запуске playbook.

### match: strict

Вариант ```match: strict``` не требует, чтобы объект был в точности как указано в задаче, но, команды, которые указаны в списке lines, должны быть в том же порядке.

Если указан список parents, команды в списке lines должны идти сразу за командами parents.

На маршрутиазаторе такой ACL:
```
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit icmp any any
 deny   ip any any
```

{% raw %}
Playbook 9_ios_config_match_strict.yml:
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
        match: strict
        provider: "{{ cli }}"
```
{% endraw %}

Выполнение playbook:
```
$ ansible-playbook 9_ios_config_match_strict.yml -v
```
![6h_ios_config_match_strict]({{ book.ansible_img_path }}6h_ios_config_match_strict.png)


Так как изменений не было, ACL остался таким же.

В такой же ситуации, при использовании ```match: exact```, было бы обнаружено изменение и ACL бы состоял только из строк в списке lines.


### match: none

Использование ```match: none``` отключает идемпотентность задачи: каждый раз при выполнении playbook, будут отправляться команды, которые указаны в задаче.

{% raw %}
Пример playbook 9_ios_config_match_none.yml:
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
        match: none
        provider: "{{ cli }}"
```
{% endraw %}

Каждый раз при запуске playbook результат будет таким:
```
$ ansible-playbook 9_ios_config_match_none.yml -v
```
![6h_ios_config_match_none]({{ book.ansible_img_path }}6h_ios_config_match_none.png)


Использование ```match: none``` подходит в тех случаях, когда, независимо от текущей конфигурации, нужно отправить все команды.

