# Prometheus-Grafana


 # Запустить контейнер:
`docker-compose -f .[место положение]/docker-compose-monitor.yml up -d`
 # Удалить контейнер:
docker-compose -f /[место положение]/docker-compose-monitor.yml down
 # Перезагрузите контейнер:
docker restart id

 #Закройте selinux
setenforce 0
nano /etc/sysconfig/selinux

 #Настройка iptables
 # Удалите собственный брандмауэр
systemctl stop firewalld.service
systemctl disable firewalld.service
 
 # Установить iptables
yum install -y iptables-services
 
 # Конфигурация nano /etc/sysconfig/iptables
```
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [24:11326]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9090 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3000 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9093 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9100 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
 ```
 #запускать 
systemctl restart iptables.service
systemctl enable iptables.service
____________________

#Отдельные команды для запуска каждого контейнера.
# Начать прометей
docker run -d -p 9090:9090 --name=prometheus \
-v /usr/local/src/config/prometheus.yml:/etc/prometheus/prometheus.yml \
-v /usr/local/src/config/node_down.yml:/etc/prometheus/node_down.yml \
prom/prometheus
 
 # Запустить графана
docker run -d -p 3000:3000 --name=grafana grafana/grafana
 
 # Запустить контейнер alertmanager
docker run -d -p 9093:9093 -v /usr/local/src/config/config.yml:/etc/alertmanager/config.yml --name alertmanager prom/alertmanager
 
 # Начать экспорт узлов
docker run -d \
  -p 9100:9100 \
  -v "/:/host:ro,rslave" \
  --name=node_exporter \
  quay.io/prometheus/node-exporter \
  --path.rootfs /host
 
 # Начать cadvisor
docker run                                    \
--volume=/:/rootfs:ro                         \
--volume=/var/run:/var/run:rw                 \
--volume=/sys:/sys:ro                         \
--volume=/var/lib/docker/:/var/lib/docker:ro  \
--publish=8080:8080                           \
--detach=true                                 \
--name=cadvisor                               \
google/cadvisor:latest





