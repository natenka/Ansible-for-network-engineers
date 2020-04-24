ios_vlans
---------

Playbook 5_ios_vlans.yml

::

    - name: Collect IOS facts
      hosts: 192.168.100.100

      tasks:

        - name: Configure vlans
          ios_vlans:
            config:
              - name: Vlan_10
                vlan_id: 10
              - name: Vlan_20
                vlan_id: 20
            state: merged
          register: result

        - name: Show result
          debug: var=result

Вывод до:

::

    SW1#sh vlan br

    VLAN Name                             Status    Ports
    ---- -------------------------------- --------- -------------------------------
    1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3
                                                    Et1/0, Et1/1, Et1/2, Et1/3
                                                    Et2/0, Et2/1, Et2/2, Et2/3
                                                    Et3/0, Et3/1, Et3/2, Et3/3
    10   VLAN0010                         active
    1002 fddi-default                     act/unsup
    1003 token-ring-default               act/unsup
    1004 fddinet-default                  act/unsup
    1005 trnet-default                    act/unsup

После:

::

    SW1#sh vlan br

    VLAN Name                             Status    Ports
    ---- -------------------------------- --------- -------------------------------
    1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3
                                                    Et1/0, Et1/1, Et1/2, Et1/3
                                                    Et2/0, Et2/1, Et2/2, Et2/3
                                                    Et3/0, Et3/1, Et3/2, Et3/3
    10   Vlan_10                          active
    20   Vlan_20                          active
    1002 fddi-default                     act/unsup
    1003 token-ring-default               act/unsup
    1004 fddinet-default                  act/unsup
    1005 trnet-default                    act/unsup

Выполнение playbook:

::

    $ ansible-playbook 5_ios_vlans.yml

    PLAY [Collect IOS facts] ***************************************************

    TASK [Configure vlans] *****************************************************
    changed: [192.168.100.100]

    TASK [Show result] *********************************************************
    ok: [192.168.100.100] => {
        "result": {
            "after": [
                {
                    "mtu": 1500,
                    "name": "default",
                    "shutdown": "disabled",
                    "state": "active",
                    "vlan_id": 1
                },
                {
                    "mtu": 1500,
                    "name": "Vlan_10",
                    "shutdown": "disabled",
                    "state": "active",
                    "vlan_id": 10
                },
                {
                    "mtu": 1500,
                    "name": "Vlan_20",
                    "shutdown": "disabled",
                    "state": "active",
                    "vlan_id": 20
                }
            ],
            "before": [
                {
                    "mtu": 1500,
                    "name": "default",
                    "shutdown": "disabled",
                    "state": "active",
                    "vlan_id": 1
                },
                {
                    "mtu": 1500,
                    "name": "VLAN0010",
                    "shutdown": "disabled",
                    "state": "active",
                    "vlan_id": 10
                }
            ],
            "changed": true,
            "commands": [
                "vlan 10",
                "name Vlan_10",
                "vlan 20",
                "name Vlan_20"
            ],
            "failed": false
        }
    }

    PLAY RECAP ******************************************************************
    192.168.100.100: ok=2  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0


