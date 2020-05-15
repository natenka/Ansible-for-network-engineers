## Вопросы и ответы по Ansible

### Как отфильтровать из списка словарей определенный словарь

```
---
- name: Collect IOS facts
  hosts: 192.168.100.100

  tasks:

    - name: Facts
      ios_facts:
        gather_subset: min
        gather_network_resources:
          - l2_interfaces

    - name: Show ansible_network_resources
      debug:
        msg: "{{ (ansible_network_resources.l2_interfaces |  selectattr('name', 'equalto', 'Ethernet1/1') | list | first ) }}"
```

Результат:
```
PLAY [Collect IOS facts] ****************************************************************

TASK [Facts] ****************************************************************************
ok: [192.168.100.100]

TASK [Show ansible_network_resources] ***************************************************
ok: [192.168.100.100] => {
    "msg": {
        "name": "Ethernet1/1",
        "trunk": {
            "encapsulation": "dot1q"
        }
    }
}

PLAY RECAP ******************************************************************************
192.168.100.100            : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Если значение в переменной
```
---

- name: Collect IOS facts
  hosts: 192.168.100.1

  vars:
    my_interface: "Ethernet0/2"

  tasks:

    - name: Facts
      ios_facts:
        gather_subset: min
        gather_network_resources:
          - l2_interfaces

    - name: Show ansible_network_resources
      debug:
        msg: "{{ (ansible_network_resources.l2_interfaces |  selectattr('name', 'match', my_interface ) | list ) }}"

```

[Примеры](https://ctrlnotes.com/how-i-used-complex-jinja2-filters-in-ansible/#)

### Ждать выполнения долгой команды и указать когда статус fail

```
---
- name: Run show commands on routers
  hosts: 192.168.100.1

  tasks:

    - name: run show commands
      ios_command:
        commands: ping 192.168.100.5 timeout 1 repeat 10
      vars:
          ansible_command_timeout: 3000
      register: result
      failed_when: "'Success rate is 100 percent' not in result.stdout"
```

### Как записать из Python данных инвентарный файл и избавиться от null

[Взято отсюда](https://stackoverflow.com/questions/37200150/can-i-dump-blank-instead-of-null-in-yaml-pyyaml)

```
In [1]: import yaml

In [2]: def represent_none(self, _):
   ...:     return self.represent_scalar('tag:yaml.org,2002:null', '')
   ...:
   ...: yaml.add_representer(type(None), represent_none)

In [3]: d = {"cisco-switches": {"hosts": {"sw1": None, "sw2": None}}}

In [4]: with open("result.yaml", "w") as f:
   ...:     yaml.dump(d, f)
   ...:

In [5]: cat result.yaml
cisco-switches:
  hosts:
    sw1:
    sw2:
```
