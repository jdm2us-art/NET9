## Задание 1. 

Лабораторная работа "Выбор и настройка протокола из семейства FHRP"

Уточнение - оборудование Cisco.

Выбрать протокол из семейства FHRP и обосновать свой выбор.
Построить топологию в Сisco Packet Tracer.
Настроить оборудование центрального офиса и филиала.
Проверить работу резервирования связи с филиалом. Для этого отключить основной канал, проверить командами ping и tracert доступность ПК в филиале и путь до него.
На вопрос 1 - ответ в свободной форме, в текстовом виде.
На вопросы 2 и 3 - приложить .pkt файл.
На вопрос 4 - скриншоты выполнения команд ping и tracert.

## ОТВЕТ:

### Конфигурация с HSRP

Конфигурация R-HQ-Main (Центр, основной роутер - Active)

enable \
configure terminal \
hostname R-HQ-Main \
no ip domain-lookup

! Настройка интерфейса в локальную сеть с HSRP \
interface gig0/0 \
 ip address 192.168.0.1 255.255.255.0 \
 standby 1 ip 192.168.0.254 \
 standby 1 priority 150 \
 standby 1 preempt \
 standby 1 track gig0/1 30 \
 no shutdown \
 exit

! Интерфейс в основной канал \
interface gig0/1 \
 ip address 10.0.1.1 255.255.255.252 \
 no shutdown \
 exit

! OSPF для маршрутизации между офисами \
router ospf 1 \
 network 192.168.0.0 0.0.0.255 area 0 \
 network 10.0.1.0 0.0.0.3 area 0 \
 default-information originate \
 exit

! Маршрут по умолчанию (для выхода в интернет, если нужно) \
ip route 0.0.0.0 0.0.0.0 192.168.0.254 

end \
write memory 

Конфигурация R-HQ-Backup (Центр, резервный роутер - Standby) 

enable \
configure terminal \
hostname R-HQ-Backup \
no ip domain-lookup

! Настройка интерфейса в локальную сеть с HSRP
interface gig0/0 \
 ip address 192.168.0.2 255.255.255.0 \
 standby 1 ip 192.168.0.254 \
 standby 1 priority 100 \
 standby 1 preempt \
 standby 1 track gig0/1 30 \
 no shutdown \
 exit

! Интерфейс в резервный канал \
interface gig0/1 \
 ip address 10.0.2.1 255.255.255.252 \
 no shutdown \
 exit

! OSPF для маршрутизации между офисами \
router ospf 1 \
 network 192.168.0.0 0.0.0.255 area 0 \
 network 10.0.2.0 0.0.0.3 area 0 \
 default-information originate \
 exit

! Маршрут по умолчанию \
ip route 0.0.0.0 0.0.0.0 192.168.0.254

end \
write memory

Конфигурация R-Branch (Филиал) - без изменений

enable \
configure terminal \
hostname R-Branch \
no ip domain-lookup 
 
interface gig0/0 \
 ip address 192.168.1.1 255.255.255.0 \
 no shutdown \
 exit

interface gig0/1 \
 ip address 10.0.1.2 255.255.255.252 \
 no shutdown \
 exit

interface gig0/2 \
 ip address 10.0.2.2 255.255.255.252 \
 no shutdown \
 exit

! OSPF - оба канала в OSPF \
router ospf 1 \
 network 192.168.1.0 0.0.0.255 area 0 \
 network 10.0.1.0 0.0.0.3 area 0 \
 network 10.0.2.0 0.0.0.3 area 0 \
 exit

! Floating Static Route для выбора дешевого канала \
ip route 192.168.0.0 255.255.255.0 10.0.1.1 \
ip route 192.168.0.0 255.255.255.0 10.0.2.2 50

end \
write memory 

Конфигурация SW-HQ (Центральный коммутатор) - без изменений

enable \
configure terminal \
hostname SW-HQ

interface fastEthernet 0/1 \
 switchport mode access \
 exit 

interface gig0/1 \
 switchport mode access \
 exit

interface gig0/2 \
 switchport mode access \
 exit

end \
write memory \

Конфигурация SW-Branch (Коммутатор филиала) - без изменений 

enable \
configure terminal \
hostname SW-Branch

interface fastEthernet 0/1 \
 switchport mode access \
 exit 

interface gig0/1 \
 switchport mode access \
 exit

end \
write memory \

Настройка ПК (изменения только для PC-HQ)

PC-HQ:

IP Address: 192.168.0.10

Subnet Mask: 255.255.255.0

Default Gateway: 192.168.0.254 (виртуальный IP HSRP!)

PC-Branch:

IP Address: 192.168.1.10

Subnet Mask: 255.255.255.0

Default Gateway: 192.168.1.1
