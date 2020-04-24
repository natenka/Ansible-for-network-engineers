ios_vlans
---------

::

    $ ansible-playbook 5_ios_vlans_.yml

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


