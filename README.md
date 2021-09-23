# Test
Реализация доступа из внешней сети с помощью Linux-шлюза и утилиты iptables

`iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 8001 -j DNAT --to-destination 10.0.0.2:80  
iptables -A FORWARD -m tcp -p tcp -d 10.0.0.2 --dport 80 -j ACCEPT  
iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 44301 -j DNAT --to-destination 10.0.0.2:443  
iptables -A FORWARD -m tcp -p tcp -d 10.0.0.2 --dport 443 -j ACCEPT  `

`iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 8002 -j DNAT --to-destination 10.0.0.3:80
iptables -A FORWARD -m tcp -p tcp -d 10.0.0.3 --dport 80 -j ACCEPT
iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 44302 -j DNAT --to-destination 10.0.0.3:443
iptables -A FORWARD -m tcp -p tcp -d 10.0.0.3 --dport 443 -j ACCEPT`

`iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 2203 -j DNAT --to-destination 10.0.1.1:22
iptables -A FORWARD -m tcp -p tcp -d 10.0.1.1 --dport 22 -j ACCEPT`

`iptables -t nat -A PREROUTING -d 1.1.1.1/32 -p tcp -m tcp --dport 2204 -j DNAT --to-destination 10.0.1.2:22
iptables -A FORWARD -m tcp -p tcp -d 10.0.1.2 --dport 22 -j ACCEPT`

Недостаток метода заключается в том, что пользователю необходимо будет знать номер внешнего порта для обращения к нужному хосту. Например, для входа на apache.example.net через HTTP нужно будет ввести адрес 1.1.1.1:8001
