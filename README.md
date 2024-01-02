# GB_introdaction_LAN

# Урок 6. Основы компьютерных сетей. Транспортный уровень. UDP и TCP.

1. Работа с программой Wireshark
2. Использование программы на python для эмитации чата
3. Настройка виртуальной машины в сети на Yandex Cloud
4. Проверка установления соединения

# Домашнее задание №6
1. Напишите свою программу сервер и запустите её ([server.py](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/server.py), [client.py](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/client.py))
2. Запустите несколько клиентов. Сымитируйте чат.
3. Подготовить код написанного сервера (сервер и клиент использовал предложенный на семинаре).
4. Отследите сокеты с помощью команды netstat. Предоставить скриншот именно сокетов чата.
5. Перехватить трафик своего чата в Wireshark и сшить сессию. Подготовить скриншот сшитой сессии с диалогом.

# Решение

1. [x] Подключаемся к виртуальной машине в облаке Яндекс или любой другой по ssh.

![ssh connect](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/0.jpg)

2. [x] Cоздаем файл 
```bash
nano server.py 
```
копируем код программы в терминал, нажимаем ctrl + X, затем Y, после чего подтверждаем сохранение в файл server.py

![Подготовка server.py](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/1.jpg)

<details>
    <summary>Возможные ошибки при подключении к серверу</summary>
В зависимости от настроек виртуальной машины могут быть проблемы с запуском сервера, уточните как настроен NAT в вашем хостинге. У меня Yandex Cloud, там необходимо в адресе сервера писать внутрений адрес IP и дальше он будет маршрутизироваться через One-to-One NAT на Публичный IP присвоенный ВМ.
</details>

3. [x] Запускаем файл для сервера
Предварительно необходимо разрешить файлу запуск, для этого необходимо выполнить команду:
```bash
chmod +x server.py 
```
![chmod](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/2.jpg)

```bash
./server.py
```
если ошибок нет мы увидим, что сервер в статусе LISTEN

![server LISTEN](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/3.jpg)

4. [x] Проверяем работу портов, с помощью команд:

```bash
ss -tulpan | grep LIST
ss -tulpan | grep ESTAB
```

![Порты](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/9.jpg)

<details>
    <summary>Описание команд bash</summary>
ss -tulpan | grep LIST - фильтруем по регулярным выражениям только сокеты listen
ss -tulpan | grep ESTAB- фильтруем по регулярным выражениям только сокеты established

tu - порты TCP UDP
l - все сокеты в состояни и listen
p - выводить id процесса
a - все сокеты ip v4, v6
n - не резолвить ip адреса в имена доменов
</details>

5. [x] Запускаем программу на стороне клиентов, IP указываем именно Публичный адрес для ВМ, запускаем одного клиента, потом второго, во втором выводе мы видим что подключился *user*.

![Клиент 1](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/5.jpg)

![Клиент 2](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/6.jpg)

так же видим, что на сервере пользователи появились в порядке подключения:

![Активность пользователей](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/4.jpg) 

6. [x] Запускаем переписку меду пользователями

![gleb](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/7.jpg)

![user](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/8.jpg)

7. [x] Перехватываем сообщения чата программой wireshark

![TCP](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/10.jpg)

8. [x] Просматриваем подключения на клиенте командой **netstat**

![netstat](https://github.com/gleb-erokhin/GB_introdaction_LAN/blob/6th/11.jpg)

