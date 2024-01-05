# GB_introdaction_LAN

# Урок 8. Основы HTTP/HTTPS и DNS

1. Работа с программой CPT
2. Настройка NAT, PAT
3. Работа с настройкой сертификата для OpenVPN

# Домашнее задание №7
1. [x] Доделать свой сертификат (скиньте его скриншот)
Настроить сеть согласно информации на схеме
(https://disk.yandex.ru/d/Vaxkf2X0RG9NGw .)
Сымитировать "Интернет" с помощью OSPF. Приватных сетей в маршрутизации быть не должно.
2. [x] Для компьютеров из Office 1 предоставить доступ в "Интернет" с помощью PAT. Для компьютеров из Office 1 должны открываться разные сайты по HTTP и HTTPS
3. [x] Открыть доступ из "Интернета" к серверам из Office 2 c помощью Port Forwarding. Для компьютеров из Office 2 по одному доменному имени.

Предоставить скриншоты открытых разных сайтов по одному доменному имени.
Предоставить скриншот таблицы NAT трансляций с Router3.
Предоставить скриншот таблицы маршрутизации с Router0.

# Решение

## Задача 1

1. [x] Для установки сертификата для openVPN неободимо получить доменное имя.
Сделем это с помощью ресурса noip. Регистриуем аккаунт, без оплаты доступно 1 доменное имя. Нажимаем добавить имя, выбираем любой домен из списка на котором будет наше доменное имя 3 уровня, указываем сразу белый IP сервера выданный ВМ, так как по умолчанию система прописывает ваш белый IP.

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/1.jpg)

2. [x] Проверяем работу доменного имени, переходим по нему

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/2.jpg)

3. [x] Для настройки сертификата неободимо использовать ресурс который предоставляет свои сертификаты Let's encrypt.com. Сертификат можно установить через ssh с помощью ресурса Certbot.

4. [x] Переходим на certbot.eff.org в строке my http используем SoftWare -> Other, OS -> Ubuntu (ваша версия), прокрутив страницу ниже увидим пошаговую инструкцию по настройке сертификата.

В нашей настройке есть нюансы, openVPN работает по https и на своем порту, это надо будет изменить в инструкции вручную

<details>
    <summary>Пункт 1 инструкции</summary>

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/3.jpg)

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/4.jpg)

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/5.jpg)
</details>

Пункт 2 инструкции мы пропускаем, так как в 22 Ubuntu snap исключен, 

Пункт 3 инструкции так же пропускаем, но если ранее snap был установлен, то его надо удалить. 

Пункт 4 инструкции будем ставить **certbot** через **apt**, данная строка ненужна

```bush
sudo apt install certbot
```
![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/6.jpg) 

Пункт 5 инструкции, нужна для создания ссылки чтобы в командной строке вызывать certbot, данная команда выполняется автоматически и указана если что-то пойдет не так и ссылка не будет работать.

```
в команде прописываем cert нажимаем TAB если команда дописалась, значит путь автоматически верно прописался.
```

Пункт 6 инструкции, используем данную строку если у нас нет веб сервера на 80 порту, нам это подходдит так как веб сервера у нас нет, после запуска настройки сертификата будет запрошен email

```bash
sudo certbot certonly --standalone
```

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/7.jpg)

Соглашаемся с политикой отказываемся от рассылок, после чего пишем наше доменное имя которое мы будем привязывать к сертификату, после обработки запроса получаем наши сертификаты

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/8.jpg)

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/9.jpg)

Пункт 7 инструкции, устанавливаем сертификаты на сервер. Сделать это можно через админку openVPN или через ssh консольное подключение Используем инструкцию с сайта OpenVPN. В кавычки вставляем свой путь к сертификату, не забываем писать sudo в начале команды если вы работаете не под root.
 
```bash
sacli --key "cs.priv_key" --value_file "/etc/letsencrypt/live/mytestgbvpn.hopto.org/privkey.pem" ConfigPut
```

```bash
sacli --key "cs.cert" --value_file "/etc/letsencrypt/live/mytestgbvpn.hopto.org/fullchain.pem" ConfigPut
```

Чтобы все изменения применились необходимо выполнить команду

```bash
sacli start
```

Мы видим что сертификаты установлены, ошибо нет.

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/10.jpg)

Теперь заходим на наш OpenVPN сервер по доменному имени https проверить работу сертификата.

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/11.jpg)

![Сертификат](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/12.jpg)

## Задача 2

1. [x] Настраиваем конфигурацию портов, включаем порты на коммутаторах, прописываем в настройках компьютеров клиентов DNS сервер.

![NAT](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/13.jpg)

2. [x] Настраиваем протокол OSPF на коммутаторах иметируя интернет

<details>
    <summary>Настройка OSPF</summary>

```
Router3
Router#conf t
Router(config)#router ospf 888
Router(config-router)#router-id 9.9.9.9
Router(config-router)#network 11.22.33.0 0.0.0.127 area 0 
Router(config-router)#network 192.168.0.0 0.0.0.255 area 0
```

```
Router0
Router#conf t
Router(config)#router ospf 888
Router(config-router)#router-id 1.1.1.1
Router(config-router)#network 11.22.33.0 0.0.0.127 area 0 
```

```
Router0
Router#conf t
Router(config)#router ospf 888
Router(config-router)#router-id 1.1.1.1
Router(config-router)#network 11.22.33.128 0.0.0.127 area 0 
```

```
Router0
Router#conf t
Router(config)#router ospf 888
Router(config-router)#router-id 1.1.1.1
Router(config-router)#network 11.22.34.0 0.0.0.255 area 0 
```

```
Router2
Router#conf t
Router(config)#router ospf 888
Router(config-router)#router-id 6.6.6.6
Router(config-router)#network 11.22.33.128 0.0.0.127 area 0
Router(config-router)#network 8.8.8.0 0.0.0.255 area 0
```

```
Router1
Router#conf t
Router(config)#router ospf 888
Router(config-router)#router-id 3.3.3.3
Router(config-router)#network 11.22.34.0 0.0.0.255 area 0 
Router(config-router)#network 10.0.0.0 0.0.0.255 area 0
```
</details>

Проверяем работу OSPF, есть все необходимые соседи и все необходимые маршруты

![NAT](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/14.jpg)

![NAT](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/15.jpg)

3. [x] Настраиваем PAT для сети 1. Для этого берем коммутатор в сети 1, Router3 ID - 9.9.9.9 и настраиваем на ней коммутацию.

```
Router(config)#interface gigabitEthernet 0/0/1 
Router(config-if)#ip nat inside 
Router(config-if)#interface gigabitEthernet 0/0/0
Router(config-if)#ip nat outside 
Router(config)#ip access-list standard NET_198
Router(config-std-nacl)#permit 192.168.0.0 0.0.0.255
Router(config-std-nacl)#ex
Router(config)#ip nat inside source list NET_198 interface gigabitEthernet 0/0/0 overload
```

4. [x] Настраиваем Port Forwarding на Router 1 на публичный адрес 11.22.34.2

```
Router(config)#interface gigabitEthernet 0/0/1 
Router(config-if)#ip nat inside 
Router(config-if)#interface gigabitEthernet 0/0/0
Router(config-if)#ip nat outside
Router(config)#ip nat inside source static tcp 10.0.0.2 80 11.22.34.2 80
Router(config)#ip nat inside source static tcp 10.0.0.3 443 11.22.34.2 443
```

5. [x] Настраиваем DNS на доменное имя и публичный адрес 11.22.34.2

![NAT](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/17.jpg)

6. [x] Проверяем работу сайта по единому доменному имени, а так же смотрим таблицу nat трансляции

![NAT](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/8th/16.jpg)
