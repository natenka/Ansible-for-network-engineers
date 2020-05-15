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
