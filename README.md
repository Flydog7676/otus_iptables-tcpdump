# otus_iptables-tcpdump
1. TCPDUMP
- установить tcpdump ```yum install tcpdump```
- просмотр активных сетевых соединений ```tcpdump -D```
- проверим всю информацию которая проходит через наш сетевой интерфейс ``` tcpdump -nn -i eth0 -c 10 ```
ключ -nn - преобразование имя хоста в цифровые адреса
ключ -i eth0 - интерфейс на котором смлтреть
ключ -c 10 - вывод 10 строк
вывод 
```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
15:57:26.503091 IP 10.0.2.15.22 > 10.0.2.2.58468: Flags [P.], seq 2913104326:2913104362, ack 277667735, win 37960, length 36
15:57:26.503602 IP 10.0.2.2.58468 > 10.0.2.15.22: Flags [.], ack 36, win 65535, length 0
15:57:26.505695 IP 10.0.2.15.22 > 10.0.2.2.58468: Flags [P.], seq 36:112, ack 1, win 37960, length 76
15:57:26.506690 IP 10.0.2.2.58468 > 10.0.2.15.22: Flags [.], ack 112, win 65535, length 0
15:57:26.507175 IP 10.0.2.15.22 > 10.0.2.2.58468: Flags [P.], seq 112:156, ack 1, win 37960, length 44
15:57:26.507806 IP 10.0.2.2.58468 > 10.0.2.15.22: Flags [.], ack 156, win 65535, length 0
15:57:26.508759 IP 10.0.2.15.22 > 10.0.2.2.58468: Flags [P.], seq 156:232, ack 1, win 37960, length 76
15:57:26.509345 IP 10.0.2.15.22 > 10.0.2.2.58468: Flags [P.], seq 232:284, ack 1, win 37960, length 52
15:57:26.509405 IP 10.0.2.2.58468 > 10.0.2.15.22: Flags [.], ack 232, win 65535, length 0
15:57:26.509663 IP 10.0.2.2.58468 > 10.0.2.15.22: Flags [.], ack 284, win 65535, length 0
```
Из дампа видно что :
  с адреса IP 10.0.2.15 ( вирт машина ) с порта 22 идет информация на 10.0.2.2 на порт 58468 и наоборот.
  Флаги соединения [P.] ответ передачи данных и [.] продление соединения . Для получения большей информации необходимо 
  указать ключ -vv
3. IPTABLES
- Используем CentOs7,он использует свой фаервол не полностью совместимый поэтому делаем 
```
systemctl disable firewalld
yum install iptables-services
systemctl enable iptables
```
- для начальной проверки состояния  фаервола ``` iptables -L ```
- Видим что все ветки пустые 
```
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
```
- Вносим правила в таблицу INPUT
``` 
iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp -j DROP
```
Первой строчкой разрешаем дочернии соединения далее разрешаем пинг системы, потом разрешаем SSH 22 порт по tcp
и запрещаем все входящие соединени tcp. Так как iptables работает последовательно по списку то у нас получается -
сначала проверяется дочерний ли это процесс, потом проверяется пинг ли это, и не идет ли пакет на 22 порт. Если любое
из этих предположений верно, то разрешается пакет, ели нет то последний пунк отрезает все отстальное.
- для сохранения таблицы фаервола делаем ``` service iptables save ``` настройки сохраняются в ```/etc/sysconfig/iptables```
- для переноса правил на другой сервер можно сделать ``` iptables-save > iptables-export```



