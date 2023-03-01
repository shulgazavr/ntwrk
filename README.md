# Разворачиваем сетевую лабораторию.

### Построить следующую архитектуру

Сеть `office1`:
- 192.168.2.0/26 - dev
- 192.168.2.64/26 - test servers
- 192.168.2.128/26 - managers
- 192.168.2.192/26 - office hardware

Сеть `office2`:
- 192.168.1.0/25 - dev
- 192.168.1.128/26 - test servers
- 192.168.1.192/26 - office hardware

Сеть `central`:
- 192.168.0.0/28 - directors
- 192.168.0.32/28 - office hardware
- 192.168.0.64/26 - wifi
```
Office1 ---\
            ----> Central --IRouter --> internet
Office2----/
```
Итого должны получится следующие сервера:
- inetRouter
- centralRouter
- office1Router
- office2Router
- centralServer
- office1Server
- office2Server

### Теоретическая часть.
- Найти свободные подсети
- Посчитать сколько узлов в каждой подсети, включая свободные
- Указать broadcast адрес для каждой подсети
- проверить нет ли ошибок при разбиении

Сеть `office1`:
- 192.168.2.0/26 - 192.168.2.{1-62} + broadcast 192.168.2.63. Хостов: 62
- 192.168.2.64/26 - 192.168.2.{65-126} + broadcast 192.168.2.127. Хостов: 62
- 192.168.2.128/26 - 192.168.2.{129-190} + broadcast 192.168.2.191. Хостов: 62
- 192.168.2.192/26 - 192.168.2.{193-254} + broadcast 192.168.2.255. Хостов: 62
```
Итого:
В данной сети, свободных подсетей нет.
Общее количество хостов - 248.
Ошибок нет.
```
Сеть `office2`:
- 192.168.1.0/25 - 192.168.1.{1-126} + broadcast 192.168.1.127. Хостов: 126
- 192.168.1.128/26 - 192.168.1.{129-190} + broadcast 192.168.1.191. Хостов: 62
- 192.168.1.192/26 - 192.168.1.{193-254} + broadcast 192.168.1.255. Хостов: 62
```
Итого:
В данной сети, свободных подсетей нет.
Общее количество хостов - 250.
Ошибок нет.
```

Сеть `central`:
- 192.168.0.0/28 - 192.168.0.{1-14} + broadcast 192.168.0.15. Хостов: 14
- 192.168.0.32/28 - 192.168.0.{33-46} + broadcast 192.168.0.47. Хостов: 14
- 192.168.0.64/26 - 192.168.0.{65-126} + broadcast 192.168.0.127. Хостов: 62
```
Итого:
В данной сети, свободны cледующие сети:
- 192.168.0.16/28. Хостов: 14
- 192.168.0.48/28. Хостов: 14
- 192.168.0.128/25. Хостов: 126
Общее количество хостов в сети - 90. 
Общее количество свободных хостов - 154. 
Ошибок нет.
```



### Практическая часть:
1. Соединить офисы в сеть согласно схеме и настроить роутинг
2. Все сервера и роутеры должны ходить в инет черз inetRouter
3. Все сервера должны видеть друг друга
4. У всех новых серверов отключить дефолт на нат (eth0), который вагрант поднимает для связи
При нехватке сетевых интервейсов добавить по несколько адресов на интерфейс

Формат сдачи ДЗ - vagrant + ansible
