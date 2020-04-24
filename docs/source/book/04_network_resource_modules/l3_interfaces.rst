ios_l3_interfaces
-----------------


Playbook:

::

    - name: Collect IOS facts
      hosts: 192.168.100.1

      vars:
        l3_intf: "{{ lookup('file', '192.168.100.1_l3_intf.json') | from_json }}"

      tasks:

        - name: Read data from file
          ios_l3_interfaces:
            config: "{{ l3_intf }}"
            state: deleted
          register: result

        - name: Show result
          debug: var=result


Файл 192.168.100.1_l3_intf:

::

    [
        {
            "ipv4": [
                {
                    "address": "4.4.4.4 255.255.255.255"
                }
            ],
            "name": "loopback0"
        },
        {
            "ipv4": [
                {
                    "address": "5.5.5.5 255.255.255.255"
                }
            ],
            "name": "loopback55"
        },
        {
            "ipv4": [
                {
                    "address": "90.1.1.1 255.255.255.255"
                }
            ],
            "name": "loopback90"
        }
    ]


До выполнения playbook

::

    R1#show ip int bri
    Interface                  IP-Address      OK? Method Status                Protocol
    Ethernet0/0                192.168.100.1   YES NVRAM  up                    up
    Ethernet0/1                192.168.200.1   YES NVRAM  up                    up
    Loopback0                  4.4.4.4         YES manual up                    up
    Loopback55                 5.5.5.5         YES manual up                    up
    Loopback90                 90.1.1.1        YES manual up                    up

После выполнения playbook

::

    R1#show ip int bri
    Interface                  IP-Address      OK? Method Status                Protocol
    Ethernet0/0                192.168.100.1   YES NVRAM  up                    up
    Ethernet0/1                192.168.200.1   YES NVRAM  up                    up
    Loopback0                  unassigned      YES manual up                    up
    Loopback55                 unassigned      YES manual up                    up
    Loopback90                 unassigned      YES manual up                    up



::

    $ ansible-playbook 3_ios_l3_interfaces.yml

    PLAY [Collect IOS facts] *******************************************************

    TASK [Read data from file] *****************************************************
    changed: [192.168.100.1]

    TASK [Show result] *************************************************************
    ok: [192.168.100.1] => {
        "result": {
            "after": [
                {
                    "name": "loopback0"
                },
                {
                    "name": "loopback55"
                },
                {
                    "name": "loopback90"
                },
                {
                    "ipv4": [
                        {
                            "address": "192.168.101.1 255.255.255.0",
                            "secondary": true
                        },
                        {
                            "address": "192.168.100.1 255.255.255.0"
                        }
                    ],
                    "name": "Ethernet0/0"
                },
                {
                    "ipv4": [
                        {
                            "address": "192.168.200.1 255.255.255.0"
                        }
                    ],
                    "name": "Ethernet0/1"
                }
            ],
            "before": [
                {
                    "ipv4": [
                        {
                            "address": "4.4.4.4 255.255.255.255"
                        }
                    ],
                    "name": "loopback0"
                },
                {
                    "ipv4": [
                        {
                            "address": "5.5.5.5 255.255.255.255"
                        }
                    ],
                    "name": "loopback55"
                },
                {
                    "ipv4": [
                        {
                            "address": "90.1.1.1 255.255.255.255"
                        }
                    ],
                    "name": "loopback90"
                },
                {
                    "ipv4": [
                        {
                            "address": "192.168.101.1 255.255.255.0",
                            "secondary": true
                        },
                        {
                            "address": "192.168.100.1 255.255.255.0"
                        }
                    ],
                    "name": "Ethernet0/0"
                },
                {
                    "ipv4": [
                        {
                            "address": "192.168.200.1 255.255.255.0"
                        }
                    ],
                    "name": "Ethernet0/1"
                }
            ],
            "changed": true,
            "commands": [
                "interface loopback0",
                "no ip address",
                "interface loopback55",
                "no ip address",
                "interface loopback90",
                "no ip address"
            ],
            "failed": false
        }
    }

    PLAY RECAP *********************************************************************
    192.168.100.1: ok=2  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0

