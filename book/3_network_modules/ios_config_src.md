## src

Параметр __src__ позволяет указывать путь к файлу в котором находится конфигурация или шаблон конфигурации, которую нужно загрузить на устройство.

Этот параметр взаимоисключающий с lines (то есть, можно указывать или lines или src). Он заменяет модуль ios_template, который скоро будет удален.

### Конфигурация

Посмотрим на пример playbook 6j_ios_config_src.yml, который использует параметр src:
```yml
---

- name: Run cfg commands on router
  hosts: 192.168.100.1
  gather_facts: false
  connection: local

  tasks:

    - name: Config ACL
      ios_config:
        src: templates/acl_cfg.txt
        provider: "{{ cli }}"
```

В файле templates/acl_cfg.txt находится такая конфигурация:
```
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit icmp any any
 deny   ip any any
```

Удаляем на маршрутизаторе этот ACL, если он остался с прошлых разделов, и запускаем playbook:
```
$ ansible-playbook 6j_ios_config_src.yml -v
```
![6j_ios_config_src](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6j_ios_config_src.png)

Неприятная особенность параметра src в том, что не видно какие изменения были внесены.
Но, возможно, в следующих версиях Ansible это будет исправлено.

Теперь на маршрутизаторе настроен ACL:
```
R1#sh run | s access
ip access-list extended IN_to_OUT
 permit tcp 10.0.1.0 0.0.0.255 any eq www
 permit tcp 10.0.1.0 0.0.0.255 any eq 22
 permit icmp any any
 deny   ip any any
```

Если запустить playbook ещё раз, но никаких изменений не будет, так как этот параметр также идемпотентен:
```
$ ansible-playbook 6j_ios_config_src.yml -v
```
![6j_ios_config_src_2](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6j_ios_config_src_2.png)


### Шаблон Jinja2

Также, в параметре src мы можем указывать шаблон Jinja2.

Пример шаблона (файл templates/ospf.j2):
```j2
router ospf 1
 router-id {{ mgmnt_ip }}
 ispf
 auto-cost reference-bandwidth 10000
{% for ip in ospf_ints %}
 network {{ ip }} 0.0.0.0 area 0
{% endfor %}
```

Нам нужно передать две переменные, чтобы все настройки были выполнены:
* mgmnt_ip - IP-адрес, который будет использоваться как router-id
* ospf_ints - список IP-адресов интерфейсов, на которых нужно включить OSPF

Настраивать OSPF мы будем на трёх маршрутизаторах, соответственно, нам нужно иметь возможность использовать разные значения переменных для разных устройств.
Именно для таких задач и нужны файлы с переменными в каталоге host_vars.

В каталоге host_vars нужно создать файлы такие файлы (если они ещё не созданы):

Файл host_vars/192.168.100.1:
```
---

hostname: london_r1
mgmnt_loopback: 100
mgmnt_ip: 10.0.0.1
ospf_ints:
  - 192.168.100.1
  - 10.0.0.1
  - 10.255.1.1
```

Файл host_vars/192.168.100.2:
```
---

hostname: london_r2
mgmnt_loopback: 100
mgmnt_ip: 10.0.0.2
ospf_ints:
  - 192.168.100.2
  - 10.0.0.2
  - 10.255.2.2
```

Файл host_vars/192.168.100.3:
```
---

hostname: london_r3
mgmnt_loopback: 100
mgmnt_ip: 10.0.0.3
ospf_ints:
  - 192.168.100.3
  - 10.0.0.3
  - 10.255.3.3
```


Теперь мы можем создавать playbook 6j_ios_config_src_jinja.yml:
```yml
---

- name: Run cfg commands on router
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: Config OSPF
      ios_config:
        src: templates/ospf.j2
        provider: "{{ cli }}"
```

Так как Ansible сам найдет переменные в каталоге host_vars, нам не нужно никак дополнительно их указывать.
Можно сразу запускать playbook:
```
$ ansible-playbook 6j_ios_config_src_jinja.yml -v
```
![6j_ios_config_src_jinja](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6j_ios_config_src_jinja.png)

Теперь на всех маршрутизаторах настроен OSPF:
```
R1#sh run | s ospf
router ospf 1
 router-id 10.0.0.1
 ispf
 auto-cost reference-bandwidth 10000
 network 10.0.0.1 0.0.0.0 area 0
 network 10.255.1.1 0.0.0.0 area 0
 network 192.168.100.1 0.0.0.0 area 0

R2#sh run | s ospf
router ospf 1
 router-id 10.0.0.2
 ispf
 auto-cost reference-bandwidth 10000
 network 10.0.0.2 0.0.0.0 area 0
 network 10.255.2.2 0.0.0.0 area 0
 network 192.168.100.2 0.0.0.0 area 0

router ospf 1
 router-id 10.0.0.3
 ispf
 auto-cost reference-bandwidth 10000
 network 10.0.0.3 0.0.0.0 area 0
 network 10.255.3.3 0.0.0.0 area 0
 network 192.168.100.3 0.0.0.0 area 0
```


Если запустить playbook ещё раз, но никаких изменений не будет:
```
$ ansible-playbook 6j_ios_config_src_jinja.yml -v
```
![6j_ios_config_src_jinja_2](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6j_ios_config_src_jinja_2.png)

### Совмещение с другими параметрами

Параметр __src__ совместим с такими параметрами:
* backup
* config
* defaults
* save (но у самого save в Ansible 2.2 проблемы с работой) 
