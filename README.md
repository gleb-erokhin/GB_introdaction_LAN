# GB_introdaction_LAN

# Урок 7. NAT. GRE.

Работа с программой CPT
Настройка GRE, NAT, PAT

# Домашнее задание №7
1. [x] Настроить сеть согласно схеме в [файле](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/s7_homework.pkt), “Интернет” — там имитация Интернета с помощью OSPF, 

**Задача 1.** Настроить на Port Forwarding на сервера в Office 2. Server0 должен предоставлять HTTP по 80му порту, а Server1 должен предоставлять HTTPS по 443 порту. Странички должны быть разные.

**Задача 2.** Настроить PAT в Office 3 для компьютеров, чтобы они выходили в интернет под одним публичным IP адресом на Router1. Предоставить скриншот открытых страниц по HTTP и HTTPS
После чего предоставить вывод show ip nat translation c Router1.

**Задача 3.** Связать сети Office 1 и Office 4 с помощью GRE. Предоставит трейс с Laptop0 до Server2.

**Задача 4.** Доделать OpenVPN, если не успели. Предоставить скриншот публичного IP до и после подключения через VPN + скриншот вывода команды ip addr.

# Решение

1. [x] Распределяем сети согласно заданию, описываем необходимые комментарии.

![Постройка сети](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/1.jpg)

2. [x] Произведем настройку OSPF на роутерах имметирующих "интернет"

![Настройка OSPF](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/2.jpg)

## Задача 2

3. [x] Настроим PAT для компьютеров Office3

```
Настраиваем router1 -> config
заходим в интерфейс -> interface ge0/0/0, укажем что это ip nat inside
заходим в интерфейс -> interface ge0/0/1, укажем что это ip nat outside
Создадим Acc list для nat (в режиме config)
ip access-list standart NET_10 (проверка только source ip) указываем имя сети
permit 10.0.0.0 0.0.0.255 предоставим доступ всем хостам 
завершаем настройку PAT (в режиме config)
ip nat inside source list NET_10 interface ge0/0/1 overload
```

команда show ip nat translations показывает что трансляций пока нет

## Задача 1

4. [x] Настроить Port Forwarding для серверов Office 2
для того чтобы заходить на сервера по публичному адресу необходимо настроить статический NAT

```
заходим в интерфейс -> interface ge0/0/0, укажем что это ip nat inside
заходим в интерфейс -> interface ge0/0/1, укажем что это ip nat outside
далее настраиваем статический маршрут NAT 
ip nat inside source static tcp 10.0.0.2 80 6.6.6.1 80
ip nat inside source static tcp 10.0.0.3 443 6.6.6.1 443
```

также перенастроем сервера, на одном оставим HTTP а на ругом HTTPS, откорректируем страницы для вывода.

![Port Forwarding](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/3.jpg)

![Port Forwarding](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/4.jpg)

![Port Forwarding](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/5.jpg)

![Port Forwarding](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/6.jpg)

## Задача 3
5. [x] Связать сети Office 1 и Office 4 с помощью GRE. 
Работать будем с сетями

 ```
Офис 1
приватная сеть 10.1.1.0/24
публичный IP 5.5.5.1/24
```

```
создаем тунельный интерфейс
conf t -> intf tunel 555
ip adress 145.145.0.1 255.255.255.0 - адрес тунельного интерфейса Router 0
tunnel source geth 0/0/1 - внешний интерфейса Router 0
tunnel destination 7.7.7.1 - публичный IP роутера назначения Router 7
exit

необходимо добавить маршрут по умолчанию, в сеть Офиса 3 с ее адресацией
config t -> ip route 192.168.145.0 255.255.255.0 145.145.0.2
```

![tunel 555](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/7.jpg)

```
Офис 3 
приватная сеть 192.168.145.0/24
публичный IP 7.7.7.1/24
```

```
создаем тунельный интерфейс
conf t -> intf tunel 555
ip adress 145.145.0.2 255.255.255.0 - адрес тунельного интерфейса Router 7
tunnel source geth 0/2/0 - внешний интерфейсс Router 7
tunnel destination 5.5.5.1 - публичный IP роутера назначения Router 0
необходимо добавить маршрут по умолчанию, в сеть Офиса 1 с ее адресацией
config t -> ip route 10.1.1.0 255.255.255.0 145.145.0.1
```

![tunel 555](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/8.jpg)

Проверим работу связности тунеля

![tunel 555](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/9.jpg)

# Задача 4
6. [x] Настройка Open VPN

Для установки используем инстркции с официального сайта OenVPN, выполняем все по пунктам. 

<details>
    <summary>Ошибки при установке OpenVPN</summary>
Если при установке возникают ошибки доступа необбходимо перейти в режим root командой sudo su (ubuntu)
</details>

После завершения установки мы получим данные для входа администратора и клиента.
проверяем что впн активен

```bash
systemctl | grep -i openvpn
```

Заходим в админку, нам необходимо создать пользователя для работы чтобы не работать с ВПН под админом. Главное после создание перезапустить сессию чтобы применились изменения. Снимаем лишние галочки и устанавливаем пароль в MORE SETTINGS. Скачиваем профиль пользователя для подключения.

![OpenVPN](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/10.jpg)

![OpenVPN](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/11.jpg)

![OpenVPN](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/12.jpg)

![OpenVPN](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/13.jpg)

Заходим по ссылке под пользователем скачиваем приложение для пк конфиг пользователя. Подключаем конфиг и логинимся пользователем. 

![OpenVPN](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/14.jpg)

<details>
    <summary>Если у вас Yandex Cloud</summary>
Если у вас Яндекс облако, то небходимо после скачивания профайла открыть его в блокноте и заменить приватный адрес на публичный.
</details>

Устанавливаем программу openvpn, загружжаем в нее измененный профйл. Проверяем свой белый ip до подключения

![OpenVPN](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/15.jpg)

Запускаем openVPN и проверяем снова белый IP, получаем IP нашей виртуальной машины.

![OpenVPN](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/7th/16.jpg)



