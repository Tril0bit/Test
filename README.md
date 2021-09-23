### Вводные данные
Локальная сеть 10.0.0.0/8 (255.0.0.0) с выходом во внешнюю сеть Интернет через
статический белый IP адрес 1.1.1.1.
В локальной сети есть несколько сервисов к которым нужно предоставить доступ извне:
1. Сервер apache2 10.0.0.2, порты 80 (HTTP) и 443 (HTTPS), hostname
apache.example.net  
2. Сервер nginx 10.0.0.3, порты 80 (HTTP) и 443 (HTTPS), hostname nginx.example.net  
3. Linux сервер1 10.0.1.1, порт 22 (SSH), hostname server1.example.com  
4. Linux сервер2 10.0.1.2, порт 22 (SSH), hostname server2.example.com  

### Задача
Описать как реализовать доступ только к требуемым локальным сервисам из внешней
сети Интернет. Клиенты коннектятся только извне, у них нет VPN или SSH туннелей в
локальную сеть.

### Реализация доступа из внешней сети с помощью Linux-шлюза и утилиты iptables
В цепочку правил *nat* добавляем перенаправления портов и разрешающие передачу пакетов правила.

Для хоста *10.0.0.2*:  
`iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 8001 -j DNAT --to-destination 10.0.0.2:80`  
`iptables -A FORWARD -m tcp -p tcp -d 10.0.0.2 --dport 80 -j ACCEPT`  
`iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 44301 -j DNAT --to-destination 10.0.0.2:443`  
`iptables -A FORWARD -m tcp -p tcp -d 10.0.0.2 --dport 443 -j ACCEPT`  

Для хоста *10.0.0.3*:  
`iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 8002 -j DNAT --to-destination 10.0.0.3:80`  
`iptables -A FORWARD -m tcp -p tcp -d 10.0.0.3 --dport 80 -j ACCEPT`  
`iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 44302 -j DNAT --to-destination 10.0.0.3:443`  
`iptables -A FORWARD -m tcp -p tcp -d 10.0.0.3 --dport 443 -j ACCEPT`  

Для хоста *10.0.1.1*:  
`iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 2203 -j DNAT --to-destination 10.0.1.1:22`  
`iptables -A FORWARD -m tcp -p tcp -d 10.0.1.1 --dport 22 -j ACCEPT`  

Для хоста *10.0.1.2*:  
`iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 2204 -j DNAT --to-destination 10.0.1.2:22`  
`iptables -A FORWARD -m tcp -p tcp -d 10.0.1.2 --dport 22 -j ACCEPT`  

Недостаток метода заключается в том, что пользователю необходимо будет знать номер внешнего порта для обращения к нужному хосту. Например, для входа на apache.example.net через HTTP нужно будет ввести адрес 1.1.1.1:8001

### Реализация доступа из внешней сети с помощью роутера Mikrotik
Для того что бы это реализовать воспользуемся web proxy.

Включим и настроим web proxy:

`/ip proxy`  
`set enabled=yes max-cache-object-size=4096KiB` 

Далее создадим правила по которым закроем доступ по телнету и email relaying и разрешим доступ к нашим веб серверам:
`/ip proxy access` 
`add comment="Enable Http Connection" disabled=yes dst-port=80`
`add comment="SSH" disabled=yes dst-port=22`  
`add dst-host=apache.example.net dst-port=80` 
`add dst-host=nginx.example.net dst-port=80`  
`add dst-host=server1.example.com dst-port=22`  
`add dst-host=server2.example.com dst-port=22`  

`add action=deny`  

Прописываем DNS-записи:

`/ip dns static`  

`add address=10.0.0.2 name=apache.example.net`  
`add address=10.0.0.3 name=nginx.example.net`  
`add address=10.0.1.1 name=server1.example.com`  
`add address=10.0.1.2 name=server2.example.com`  

Вот вроде и все осталось только зарулить все запросы из вне на наш прокси:

/ip firewall nat

add action=redirect chain=dstnat dst-port=80 in-interface=WAN protocol=tcp to-ports=8080

И не забываем в фаерволе разрешить доступ по порту 8080:

add action=accept chain=input comment=toProxy dst-port=8080 protocol=tcp
