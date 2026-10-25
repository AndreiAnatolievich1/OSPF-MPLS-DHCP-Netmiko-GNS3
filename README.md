# Мультидоменная сеть OSPF + MPLS в GNS3

### Multi-Area OSPF · MPLS/LDP · NAT · DHCP · Python/Netmiko

Комплексный лабораторный проект: два OSPF-домена, маршрутизация между ними, MPLS/LDP на десяти маршрутизаторах, DHCP для пользовательских VLAN и автоматический сбор диагностической информации по SSH.

![GNS3](https://img.shields.io/badge/GNS3-10%20маршрутизаторов-1f6f8b?style=flat-square)
![Cisco IOS](https://img.shields.io/badge/Cisco%20IOS-OSPF%20%2B%20MPLS-24435c?style=flat-square)
![Python](https://img.shields.io/badge/Python-Netmiko-3a7d44?style=flat-square)

## Содержание

- [Архитектура проекта](#архитектура-проекта)
- [Топология](#топология)
- [Реализованные технологии](#реализованные-технологии)
- [Структура репозитория](#структура-репозитория)
- [Проверка работоспособности](#проверка-работоспособности)
- [Автоматизация Netmiko](#автоматизация-netmiko)
- [Запуск](#запуск)
- [Полученные навыки](#полученные-навыки)

## Архитектура проекта

Стенд объединяет **два OSPF-домена**. Каждый домен содержит backbone Area 0 и дополнительные области. Между доменами настроены NAT и redistribution статических маршрутов через `route-map`. В пользовательском сегменте работает DHCP для двух VLAN, а на маршрутизаторах включён MPLS с LDP.

Проект показывает не только настройку отдельных протоколов, но и их совместную работу в многоуровневой топологии, а также автоматизацию рутинного сбора данных.

## Топология

![Топология OSPF и MPLS в GNS3](https://github.com/user-attachments/assets/cca70d79-455f-450c-a5a3-a4b495b902f7)

В стенде используются маршрутизаторы **R1–R10**, коммутаторы доступа и конечные узлы. В репозитории также сохранена [видеодемонстрация подключения](config/процесс%20подключения.mp4).

## Реализованные технологии

| Технология | Применение в проекте |
|---|---|
| **Multi-Area OSPF** | Иерархическая маршрутизация в двух доменах с Area 0 и дополнительными областями |
| **Route redistribution** | Редистрибуция статических маршрутов через `route-map` |
| **MPLS + LDP** | Коммутация по меткам и формирование LDP-соседств на маршрутизаторах |
| **VLAN** | Разделение пользовательских сегментов |
| **DHCP / DHCP relay** | Централизованная выдача адресов для двух VLAN |
| **NAT + ACL** | Трансляция внутренних адресов при обмене с внешней сетью |
| **Python + Netmiko** | SSH-подключение к устройствам и сохранение диагностических команд |

## Структура репозитория

| Файл или каталог | Содержимое |
|---|---|
| [`Project_1.gns3`](Project_1.gns3) | Топология GNS3 |
| [`config/skript.md`](config/skript.md) | Python/Netmiko-скрипт сбора диагностики |
| [`config/процесс подключения.mp4`](config/процесс%20подключения.mp4) | Видеодемонстрация подключения |
| `config/router1.md` … `config/router10.md` | Сохранённые выводы OSPF, MPLS и `show running-config` |

### Диагностические снимки маршрутизаторов

| R1 | [`config/router1.md`](config/router1.md) | OSPF-соседство, MPLS forwarding table, running-config |
| R2 | [`config/router2.md`](config/router2.md) | OSPF-соседство, MPLS forwarding table, running-config |
| R3 | [`config/router3.md`](config/router3.md) | OSPF-соседство, MPLS forwarding table, running-config |
| R4 | [`config/router4.md`](config/router4.md) | OSPF-соседство, MPLS forwarding table, running-config |
| R5 | [`config/router5.md`](config/router5.md) | OSPF-соседство, MPLS forwarding table, running-config |
| R6 | [`config/router6.md`](config/router6.md) | OSPF-соседство, MPLS forwarding table, running-config |
| R7 | [`config/router7.md`](config/router7.md) | OSPF-соседство, MPLS forwarding table, running-config |
| R8 | [`config/router8.md`](config/router8.md) | OSPF-соседство, MPLS forwarding table, running-config |
| R9 | [`config/router9.md`](config/router9.md) | OSPF-соседство, MPLS forwarding table, running-config |
| R10 | [`config/router10.md`](config/router10.md) | OSPF-соседство, MPLS forwarding table, running-config |

## Проверка работоспособности

| Команда | Ожидаемый результат |
|---|---|
| `show ip ospf neighbor` | Все запланированные соседства находятся в состоянии `FULL` |
| `show ip route ospf` | В таблице присутствуют маршруты OSPF и межобластные маршруты |
| `show mpls ldp neighbor` | LDP-соседства установлены |
| `show mpls forwarding-table` | Для префиксов назначены локальные и исходящие метки |
| `show ip nat translations` | Во время обмена трафиком создаются NAT-трансляции |
| Запрос адреса клиентом | Хосты пользовательских VLAN получают параметры по DHCP |

## Автоматизация Netmiko

Скрипт из [`config/skript.md`](config/skript.md) последовательно:

1. проверяет маршрут до сети управления `10.0.0.0/24`;
2. создаёт отдельный каталог с меткой времени;
3. проверяет ICMP-доступность маршрутизаторов;
4. подключается к каждому устройству по SSH через Netmiko;
5. сохраняет результаты команд в отдельный файл маршрутизатора.

Собираемые команды:

```cisco
show ip ospf neighbor
show mpls forwarding-table
show running-config
```

Для работы скрипта нужны Python, пакет `netmiko`, сетевой доступ к management-адресам и учётная запись на устройствах. Перед запуском необходимо адаптировать адреса и учётные данные к собственному стенду.

## Запуск

1. Установите GNS3 и совместимые образы Cisco IOS.
2. Откройте `Project_1.gns3` и обновите пути к образам, если они отличаются.
3. Запустите узлы и примените сохранённые конфигурации из каталога `config`.
4. Выполните ручные проверки OSPF, MPLS, DHCP и NAT.
5. Для автоматического сбора данных перенесите листинг из `config/skript.md` в Python-файл и настройте параметры подключения.

> Учебные пароли и management-адреса приведены только для лаборатории. Не используйте их в производственной сети и не публикуйте реальные учётные данные.

## Полученные навыки

- построение и проверка Multi-Area OSPF;
- redistribution статических маршрутов через `route-map`;
- настройка DHCP relay на сабинтерфейсах VLAN;
- проверка LDP-соседств и MPLS forwarding table;
- автоматизация сбора конфигураций и диагностики с помощью Netmiko;
- документирование результатов по каждому сетевому устройству.
