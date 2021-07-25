# otus_iptables-tcpdump
1. TCPDUMP
2. IPTABLES
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

