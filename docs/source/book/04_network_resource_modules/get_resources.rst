Получение информации о ресурсах
-------------------------------

Получить структурированную информацию о ресурсах можно с помощью модулей по сбору фактов,
например, ios_facts. В модуле появился дополнительный параметр gather_network_resources,
который позволяет получить информацию о ресурсах в одинаковом виде, независимо от 
платформы.

Пример получения информация о ресурсе interfaces с Cisco IOS (1_ios_facts_network_resources.yml):

::

    - name: Collect IOS facts
      hosts: 192.168.100.1

      tasks:

        - name: Facts
          ios_facts:
            gather_subset: min
            gather_network_resources:
              - interfaces

        - name: Show ansible_network_resources
          debug: var=ansible_network_resources

Все ресурсы собираются в отдельную переменную ansible_network_resources, а уже внутри
нее каждому ресурсу создан отдельный ключ.

::

    ansible-playbook 1_ios_facts_network_resources.yml

.. figure:: https://raw.githubusercontent.com/natenka/Ansible-for-network-engineers/master/images/resources_get.png

Важный аспект получения информации о ресурсе - она одинаково структурирована
для разных платформ.
