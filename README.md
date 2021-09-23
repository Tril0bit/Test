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
В цепочку правил nat добавляем перенаправления портов и разрешающие передачу пакетов правила.

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
