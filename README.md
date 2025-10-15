# Домашнее задание к занятию 2 "`Кластеризация и балансировка нагрузки`" - `Осаковская Анна`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

`Конфигурационный файл haproxy`

```
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

    # Default SSL material locations
    ca-base /etc/ssl/certs
    crt-base /etc/ssl/private

    # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
    ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
    ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
    ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
    log global
    mode tcp
    option tcplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

listen stats  
    bind :888
    mode http
    stats enable
    stats uri /stats
    stats refresh 5s
    stats realm Haproxy\ Statistics

listen web_tcp
    mode tcp
    bind :1325
    balance roundrobin
    option tcp-check
    server s1 127.0.0.1:8888 check inter 3s fall 3 rise 2
    server s2 127.0.0.1:9999 check inter 3s fall 3 rise 2

frontend web_http
    mode http
    bind :8088
    default_backend http_servers

backend http_servers
    mode http
    balance roundrobin
    server s1 127.0.0.1:8888 check
    server s2 127.0.0.1:9999 check
```

`Скриншоты: перенаправление запросов на разные серверы при обращении к HAProxy`

`TCP балансировка L4 уровень`

<img width="738" height="552" alt="TCP L4 balancing" src="https://github.com/user-attachments/assets/8e339042-f98a-41e7-ab4a-f72561fe0ace" />

`Балансировка HTTP`

<img width="738" height="546" alt="HTTP balancing" src="https://github.com/user-attachments/assets/af441e1c-341f-4c92-a469-342dd35a02eb" />

`Общая статистика`

<img width="1595" height="679" alt="Снимок экрана 2025-10-14 в 23 21 45" src="https://github.com/user-attachments/assets/be1085a1-39e8-4195-bf81-8829cae68c84" />

---

### Задание 2

`Конфигурационный файл haproxy`

```
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

    ca-base /etc/ssl/certs
    crt-base /etc/ssl/private

    ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
    ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
    ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

listen stats
    bind :888
    mode http
    stats enable
    stats uri /stats
    stats refresh 5s
    stats realm Haproxy\ Statistics

frontend http_frontend
    mode http
    bind :8088
    
    acl is_example_local hdr(host) -i example.local
    use_backend weighted_servers if is_example_local
    default_backend default_servers

backend weighted_servers
    mode http
    balance roundrobin
    server s1 127.0.0.1:8888 weight 2 check
    server s2 127.0.0.1:8889 weight 3 check
    server s3 127.0.0.1:8890 weight 4 check

backend default_servers
    mode http
    balance roundrobin
    server s1 127.0.0.1:8888 check
    server s2 127.0.0.1:8889 check
    server s3 127.0.0.1:8890 check
```

`Скриншоты: перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него`

`Тест Round Robin с доменом example.local`

<img width="751" height="631" alt="With domain" src="https://github.com/user-attachments/assets/205b20d7-5742-491a-8601-89b0374d49b0" />

`Тест Round Robin без домена`

<img width="889" height="135" alt="Without domain" src="https://github.com/user-attachments/assets/78f1f9e6-4837-42d1-b8ab-d4d08af46900" />

`Общая статистика`

<img width="1307" height="637" alt="Снимок экрана 2025-10-15 в 11 11 56" src="https://github.com/user-attachments/assets/c24bb79e-e192-4973-8045-f377a777eea5" />

---

