## parents

Если нам нужно применить команды в каком-то подрежиме, а не в глобальном конфигурационно режиме, нужно использовать параметр parents.

Например, нам нужно применить такие команды:
```
line vty 0 4
 login local
 transport input ssh
```

В таком случае, playbook 6a_ios_config_parents_basic.yml будет выглядеть так:
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

Запуск будет выполняться аналогично предыдущим playbook:
```
$ ansible-playbook 6a_ios_config_parents_basic.yml
```

![6a_ios_config_parents_basic](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/6a_ios_config_parents_basic.png)


Если нам нужно выполнить команду в нескольких вложенных режимах, мы указываем подрежимы в списке parents.
Например, нам нужно выполнить такие команды:

```
policy-map OUT_QOS
 class class-default
  shape average 100000000 1000000
```

Тогда playbook 6a_ios_config_parents_mult.yml будет выглядеть так:
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


