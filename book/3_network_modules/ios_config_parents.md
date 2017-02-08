{% raw %}
## parents

Параметр parents используется, чтобы указать в каком подрежиме применить команды.

Например, необходимо применить такие команды:
```
line vty 0 4
 login local
 transport input ssh
```

В таком случае, playbook 2_ios_config_parents_basic.yml будет выглядеть так:
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
        provider: "{{ cli }}"

```
{% endraw %}

Запуск будет выполняться аналогично предыдущим playbook:
```
$ ansible-playbook 2_ios_config_parents_basic.yml
```

![6a_ios_config_parents_basic]({{ book.ansible_img_path }}6a_ios_config_parents_basic.png)


Если команда находится в нескольких вложенных режимах, подрежимы указываются в списке parents.

{% raw %}
Например, необходимо выполнить такие команды:
```
policy-map OUT_QOS
 class class-default
  shape average 100000000 1000000
```

Тогда playbook 2_ios_config_parents_mult.yml будет выглядеть так:
```yml
---

- name: Run cfg commands on routers
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: Config QoS policy
      ios_config:
        parents:
          - policy-map OUT_QOS
          - class class-default
        lines:
          - shape average 100000000 1000000
        provider: "{{ cli }}"
```


{% endraw %}
