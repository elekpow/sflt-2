# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки» - Левин Игорь

### Цель задания
В результате выполнения этого задания вы научитесь:
1. Настраивать балансировку с помощью HAProxy
2. Настраивать связку HAProxy + Nginx

------

### Чеклист готовности к домашнему заданию

1. Установлена операционная система Ubuntu на виртуальную машину и имеется доступ к терминалу
2. Просмотрены конфигурационные файлы, рассматриваемые на лекции, которые находятся по [ссылке](https://github.com/netology-code/sflt-homeworks/blob/main/2)


------


### Задание 1
- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy, воспользуйтесь материалами к лекции по [ссылке](https://github.com/netology-code/sflt-homeworks/blob/main/2)
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.


---

### Выполнения задания 1


Устанавливаем HAProxy c конфигурацией 
[ссылка](https://github.com/elekpow/sflt-2/blob/main/sflt-2/haproxy.cfg)

```
 sudo apt-get install haproxy
 
 sudo nano /etc/haproxy/haproxy.cfg
  
```

Cоздаем директории для двух веб-серверов http1 и http2 с индекснымим файлами index.html

Запуск двух python серверов осуществляем командами:

```
python3 -m http.server 8888 --bind 0.0.0.0

python3 -m http.server 9999 --bind 0.0.0.0

```
 
Проверка работоспособности:

![servers.JPG](https://github.com/elekpow/sflt-2/blob/main/sflt-2/servers.JPG)


в секции frontend указываем имя хоста например: myweb.com

```
frontend myweb  # секция фронтенд
        mode tcp
        bind :3306
        acl ACL_myweb.com hdr(host) -i myweb.com
        use_backend web_servers if ACL_myweb.com

```

Конфигурация haproxy для балансировки по методу Round-robin на 4 уровне.

```
backend web_servers    # секция бэкенд
        mode tcp
        balance roundrobin
        server s1 127.0.0.1:8888 check
        server s2 127.0.0.1:9999 check

```

проверка :

![hosts.JPG](https://github.com/elekpow/sflt-2/blob/main/sflt-2/hosts.JPG)

вирутальная машина зарпущена на сервисе Yandex Cloud , для проверки веб хостов , открываем порт 888

```
sudo ufw allow 888

```

HAProxy Statistics

![stat.JPG](https://github.com/elekpow/sflt-2/blob/main/sflt-2/stat.JPG)



---

### Задание 2
- Запустите три simple python сервера на своей виртуальной машине на разных портах
- Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
- HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.


---

### Выполнения задания 2

запустим три сервера : Server1 порт: 8888,Server1 порт: 9999,Server1 порт: 7777


в настройках HAProxy перенастроим  fronted и backend:

для домена "example.local"

```
frontend example # секция фронтенд
        bind :8088
        acl ACL_example.local hdr(host) -i example.local
        use_backend web_servers if ACL_example.local
```

настроим веса для каждого сервера

```
backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 check weight 2
        server s2 127.0.0.1:9999 check weight 3
        server s3 127.0.0.1:7777 check weight 4
```


HAProxy Statistics

![stat_3_servers.JPG](https://github.com/elekpow/sflt-2/blob/main/sflt-2/stat_3_servers.JPG)



Конфигурация HAProxy 
[ссылка](https://github.com/elekpow/sflt-2/blob/main/sflt-2/haproxy_2.cfg)


прверяем: все три сервера участвуют в балансировке

![testing_hosts.JPG](https://github.com/elekpow/sflt-2/blob/main/sflt-2/testing_hosts.JPG)

