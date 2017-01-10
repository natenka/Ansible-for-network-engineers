{% raw %}
## Фильтры Jinja2

Ansible позволяет использовать фильтры Jinja2 не только в шаблонах, но и в playbook.

С помощью фильтров можно преобразовывать значения переменных, переводить их в другой формат и др.

Ansible поддерживает не только встроенные фильтры Jinja, но и множество собственных фильтров.
Мы не будем рассматривать все фильтры, поэтому, если вы не найдете нужный вам фильтр тут, посмотрите [документацию](http://docs.ansible.com/ansible/playbooks_filters.html).

Мы уже использовали фильтры:
* to_nice_json в разделе [ios_facts](https://natenka.gitbooks.io/pyneng/content/book/chapter15/4b_ios_facts.html)
* regex_findall в разделе [роли](https://natenka.gitbooks.io/pyneng/content/book/chapter15/6c_roles.html)

> Если вас интересуют фильтр в контексте использования их в шаблонах, это рассматривалось в разделе [Фильтры](https://natenka.gitbooks.io/pyneng/content/book/chapter13/3d_syntax_filter.html).

Для начала, перечислим несколько фильтров для общего понимания возможностей.

Ansible поддерживает такие фильтры (список не полный):
* [фильтры для форматирования данных](http://docs.ansible.com/ansible/playbooks_filters.html#filters-for-formatting-data):
  * ```{{ var | to_nice_json }}``` - преобразует данные в формат JSON
  * ```{{ var | to_nice_yaml }}``` - преобразует данные в формат YAML
* переменные
  * ```{{ var | default(9) }}``` - позволяет определить значение по умолчанию для переменной
  * ```{{ var | default(omit) }}``` - позволяет пропустить переменную, если она не определена
* списки
  * ```{{ lista | min }}``` - минимальный элемент списка
  * ```{{ lista | max }}``` - максимальный элемент списка
* [фильтры, которые работают множествами](http://docs.ansible.com/ansible/playbooks_filters.html#set-theory-filters)
  * ```{{ list1 | unique }}``` - возвращает множество уникальных элементов из списка
  * ```{{ list1 | difference(list2) }}``` - разница между двумя списками: каких элементов первого списка нет во втором
* [фильтр для работы с IP-адресами](http://docs.ansible.com/ansible/playbooks_filters_ipaddr.html)
  * ```{{ var | ipaddr }}``` - проверяет является ли переменная IP-адресом
* регулярные выражения
  * regex_replace - замена в строке
  * regex_search - ищет первое совпадение с регулярным выражением
  * regex_findall - ищет все совпадения с регулярным выражением
* фильтры, которые применяют другие фильтры к последовательности объектов:
  * map: ```{{ list3 | map('int') }}``` - применяет другой фильтр к последовательности элементов (например, список). Также позволяет брать значение определенного атрибута у каждого объекта в списке.
  * select: ```{{ list4 | select('int') }}``` - фильтрует последовательность применяя другой фильтр к каждому из элементов. Остаются только те объекты, для которых тест отработал.
* конвертация типов
  *  ```{{ var | int }}``` - конвертирует значение в число, по умолчанию, в десятичное
  *  ```{{ var | list }}``` - конвертирует значение в список

### to_nice_yaml

Фильтры to_nice_yaml (to_nice_json) можно использовать для того, чтобы записать нужную информацию в файл.

> Ansible также поддерживает фильтры to_json и to_yaml, но их сложнее воспринимать визуально.

Повторим пример из раздела ios_facts. Playbook 8_playbook_filters_to_nice_yaml.yml:
```json
---

- name: Collect IOS facts
  hosts: cisco-routers
  gather_facts: false
  connection: local

  tasks:

    - name: Facts
      ios_facts:
        gather_subset: all
        provider: "{{ cli }}"
      register: ios_facts_result

    - name: Copy facts to files
      copy:
        content: "{{ ios_facts_result | to_nice_yaml }}"
        dest: "all_facts/{{inventory_hostname}}_facts.yml"
```

Результат выполнения playbook будет таким:
```
$ ansible-playbook 8_playbook_filters_to_nice_yaml.yml
```

![8_playbook_filters_to_nice_yaml](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/8_playbook_filters_to_nice_yaml.png)

Теперь в каталоге all_facts появились такие файлы:
```
192.168.100.1_facts.yml
192.168.100.2_facts.yml
192.168.100.3_facts.yml
```

Файл all_facts/192.168.100.1_facts.yml:
```
ansible_facts:
    ansible_net_all_ipv4_addresses:
    - 192.168.200.1
    - 192.168.100.1
    ansible_net_all_ipv6_addresses: []
    ansible_net_config: "Building configuration...\n\nCurrent configuration : 7367\
        \ bytes\n!\n! Last configuration change at 16:33:06 UTC Mon Jan 9 2017\nversion\
        \ 15.2\nno service timestamps debug uptime\nno service timestamps log uptime\n\
        service password-encryption\n!\nhostname R1\n!\nboot-start-marker\n
...
```

### regex_findall, map, max

Посмотрим пример использования фильтров одновременно и в шаблоне, и в playbook.

Сделаем playbook, который будет генерировать конфигурацию site-to-site VPN (GRE + IPsec) для двух сторон.

В этом случае, мы не будем отправлять команды на устройства, а воспользуемся модулем template, чтобы сгенерировать конфигурацию и записать её в локальные файлы.

Настройка GRE + IPsec выглядит таким образом:
```
crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 5
 hash sha

crypto isakmp key cisco address 192.168.100.2

crypto ipsec transform-set AESSHA esp-aes esp-sha-hmac
 mode transport

crypto ipsec profile GRE
 set transform-set AESSHA

interface Tunnel0
 ip address 10.0.1.2 255.255.255.252
 tunnel source 192.168.100.1
 tunnel destination 192.168.100.2
 tunnel protection ipsec profile GRE
```

Playbook 8_playbook_filters_regex.yml
```
---

- name: Cfg VPN
  hosts: 192.168.100.1,192.168.100.2
  gather_facts: false
  connection: local


  vars:
    wan_ip_1: 192.168.100.1
    wan_ip_2: 192.168.100.2
    tun_ip_1: 10.0.1.1 255.255.255.252
    tun_ip_2: 10.0.1.2 255.255.255.252


  tasks:

    - name: Collect facts
      ios_facts:
        gather_subset:
          - "!hardware"
        provider: "{{ cli }}"

    - name: Collect current tunnel numbers
      set_fact:
        tun_num: "{{ ansible_net_config | regex_findall('interface Tunnel(.*)') }}"

    #- debug: var=tun_num

    - name: Generate VPN R1
      template:
        src: templates/ios_vpn1.txt
        dest: configs/result1.txt
      when: wan_ip_1 in ansible_net_all_ipv4_addresses

    - name: Generate VPN R2
      template:
        src: templates/ios_vpn2.txt
        dest: configs/result2.txt
      when: wan_ip_2 in ansible_net_all_ipv4_addresses
```

Разберемся с содержимым playbook.
В этом playbook один сценарий и он применяется только к двум устройствам:
```
- name: Cfg VPN
  hosts: 192.168.100.1,192.168.100.2
  gather_facts: false
  connection: local
```

Наша задача была в том, чтобы сделать playbook, который  можно легко повторно использовать.
А значит, нужно сделать так, чтобы нам  не нужно было повторять несколько раз одни и те же вещи (например, адреса).

И, в данном случае не очень удобно будет, если мы будем создавать переменные в файлах host_vars. Удобней создать их в самом playbook, а когда нужно будет сгенерировать конфигурацию для другой пары устройств, достаточно будет сменить адреса в playbook.

Для этого, в сценарии создан блок с переменными:
```
  vars:
    wan_ip_1: 192.168.100.1
    wan_ip_2: 192.168.100.2
    tun_ip_1: 10.0.1.1 255.255.255.252
    tun_ip_2: 10.0.1.2 255.255.255.252
```

> Вместо адресов wan_ip_1, wan_ip_2, вам нужно будет подставить белые адреса маршрутизаторов. 

Адреса мы задаем вручную.
Но, всё остальное, хотелось бы делать автоматически.

Например, для настройки VPN нам нужно знать номер туннеля, чтобы создать интерфейс. 
Но мы не можем взять какой-то произвольный номер, так как на маршрутизаторе уже может существовать туннель с таким номером.
Нам нужно определять автоматически.

Для этого, мы сначала собираем факты об устройстве:
```
    - name: Collect facts
      ios_facts:
        gather_subset:
          - "!hardware"
        provider: "{{ cli }}"
```

Теперь мы создадим факт, для каждого из маршрутизаторов, который будет содержать список текущих номеров туннелей.
Создаем факт мы с помощью модуля set_fact.

Факт создается на основе того, что нам выдаст результат поиска в конфигурации строки ```interface TunnelX``` с помощью фильтра regex_findall.
Этот фильтр ищет все строки, которые совпадают с регулярным выражением.
А затем, запоминает и записывает в список то, что попало в круглые скобки (номер туннеля).
```
    - name: Collect current tunnel numbers
      set_fact:
        tun_num: "{{ ansible_net_config | regex_findall('interface Tunnel(.*)') }}"
```

Дальнейшая обработка списка будет выполняться в шаблоне.

Затем, мы генерируем шаблоны для устройств.
Для каждого устройства есть свой шаблон.
Поэтому, в каждой задаче стоит условие
```
      when: wan_ip_1 in ansible_net_all_ipv4_addresses
```

Благодаря этому условию, мы выбираем для какого устройства будет сгенерирован какой конфиг.

ansible_net_all_ipv4_addresses - это список IP-адресов на устройства, вида:
```
ansible_net_all_ipv4_addresses:
    - 192.168.200.1
    - 192.168.100.1
```

Этот список был получен в задаче по сбору фактов.

Задача будет выполняться только в том случае, если в списке адресов на устройстве, был найден адрес wan_ip_1.


Генерация шаблонов:
```
    - name: Generate VPN R1
      template:
        src: templates/ios_vpn1.txt
        dest: configs/result1.txt
      when: wan_ip_1 in ansible_net_all_ipv4_addresses

    - name: Generate VPN R2
      template:
        src: templates/ios_vpn2.txt
        dest: configs/result2.txt
      when: wan_ip_2 in ansible_net_all_ipv4_addresses
```

Шаблон templates/ios_vpn1.txt выглядит таким образом:
```
{% if not tun_num %}
 {% set tun_num = 0 %}
{% else %}
 {% set tun_num = tun_num | map('int') | max %}
 {% set tun_num = tun_num + 1 %}
{% endif %}

crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 5
 hash sha

crypto isakmp key cisco address {{ wan_ip_2 }}

crypto ipsec transform-set AESSHA esp-aes esp-sha-hmac
 mode transport

crypto ipsec profile GRE
 set transform-set AESSHA

interface Tunnel {{ tun_num }}
 ip address {{ tun_ip_1 }}
 tunnel source {{ wan_ip_1 }}
 tunnel destination {{ wan_ip_2 }}
 tunnel protection ipsec profile GRE
```

Шаблон templates/ios_vpn2.txt выглядит точно также, меняются только переменные с адресами:
```
{% if not tun_num %}
 {% set tun_num = 0 %}
{% else %}
 {% set tun_num = tun_num | map('int') | max %}
 {% set tun_num = tun_num + 1 %}
{% endif %}

crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 5
 hash sha

crypto isakmp key cisco address {{ wan_ip_1 }}

crypto ipsec transform-set AESSHA esp-aes esp-sha-hmac
 mode transport

crypto ipsec profile GRE
 set transform-set AESSHA

interface Tunnel {{ tun_num }}
 ip address {{ tun_ip_2 }}
 tunnel source {{ wan_ip_2 }}
 tunnel destination {{ wan_ip_1 }}
 tunnel protection ipsec profile GRE
```

В самой конфигурации никаких сложностей нет.
Обычная подстановка переменных.

Разберемся с этой частью:
```
{% if not tun_num %}
 {% set tun_num = 0 %}
{% else %}
 {% set tun_num = tun_num | map('int') | max %}
 {% set tun_num = tun_num + 1 %}
{% endif %}
```

Переменная tun_num - это факт, который мы устанавливали в playbook.
Если на маршрутизаторе созданы туннели, эта переменная содержит список номеров туннелей.
Но, если на маршрутизаторе нет ни одного туннеля, мы получим пустой список.

Если мы получили пустой список, то можно создавать интерфейс Tunnel0.
Если мы получили список с номерами, то мы вычисляем максимальный и используем следующий номер, для нашего туннеля.

Если переменная tun_num будет пустым списком, нам нужно установить её равной 0 (пустой список - False):
```
{% if not tun_num %}
 {% set tun_num = 0 %}
```

Иначе, нам нужно сначала конвертировать строки в числа, затем выбрать из чисел максимальное и добавить 1.
Это и будет значение переменной tun_num.
```
{% else %}
 {% set tun_num = tun_num | map('int') | max %}
 {% set tun_num = tun_num + 1 %}
{% endif %}
```

Выполнение playbook (создайте каталог configs):
```
$ ansible-playbook 8_playbook_filters_regex.yml
```

![8_playbook_filters_regex](https://raw.githubusercontent.com/natenka/PyNEng/master/book/chapter15/images/8_playbook_filters_regex.png)

> На маршрутизаторе 192.168.100.1 специально созданы несколько туннелей. А на маршрутизаторе 192.168.100.2 нет ни одного туннеля.

В результате, мы получили такие конфигурации (configs/result1.txt):
```

crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 5
 hash sha

crypto isakmp key cisco address 192.168.100.2

crypto ipsec transform-set AESSHA esp-aes esp-sha-hmac
 mode transport

crypto ipsec profile GRE
 set transform-set AESSHA

interface Tunnel 16
 ip address 10.0.1.1 255.255.255.252
 tunnel source 192.168.100.1
 tunnel destination 192.168.100.2
 tunnel protection ipsec profile GRE
```

Файл configs/result2.txt:
```

crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 5
 hash sha

crypto isakmp key cisco address 192.168.100.1

crypto ipsec transform-set AESSHA esp-aes esp-sha-hmac
 mode transport

crypto ipsec profile GRE
 set transform-set AESSHA

interface Tunnel 0
 ip address 10.0.1.2 255.255.255.252
 tunnel source 192.168.100.2
 tunnel destination 192.168.100.1
 tunnel protection ipsec profile GRE
```

{% endraw %}

