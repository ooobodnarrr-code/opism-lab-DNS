<!--
Бланк звіту до лабораторної роботи «DNS».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Коментарі такого вигляду в GitHub не відображаються; видаляти їх не обов'язково.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота. DNS

| Поле | Значення |
|---|---|
| ПІБ |Боднар Олександра |
| Група | 2.02|
| Ідентифікатор (`sNN`) |cs-202-02 |
| Дата виконання |27.10.2026|
| Середовище | GitHub Codespaces |
| Версія `dig` | DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu |

## Параметри варіанта

<!-- Переписати з картки stand/cards/sNN.md. -->

| Параметр | Значення |
|---|---|
| `$TTL`, с |30 |
| TTL запису SOA, с | 120|
| Поле `minimum` SOA, с | 60|
| Псевдонім (CNAME на `www`) | portal|
| Другий хост і його TTL, с | next-20|
| Пріоритет MX | 15|
| Токен TXT | 	lab-token=f71fca29|
| Ім'я для завдання 5 |	new-zfkmu.cs-202-02.lab  |

## Завдання 1. Складові URI

### 1.1. Розбір

<!-- Відсутню складову позначати «—». Порт за замовчуванням записувати з позначкою «(за замовч.)». -->

| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 |http|-| www.cs-202-02.lab| 80(за замовчуванням)| /orders/search| id=35&lang=x|- |
| 2 | http|- |portal.cs-202-02.lab |80 |/search/| id=%D0%BA%D0%B8%D1%97%D0%B2|-|
| 3 |http | user:pass|next.cs-202-02.lab|80(за замовчуванням) |/orders|-|section-2|
| 4 |http | -|www.cs-202-02.lab |4443 |/orders/../search |-|-|
| 5 |mailto| -| -| -|admin@cs-202-02.lab|subject=orders|-|

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 | www.cs-202-02.lab|www.cs-202-02.lab |/orders/search?id=35&lang=x |
| 2 | portal.cs-202-02.lab|portal.cs-202-02.lab:80 | /search/?id=%D0%BA%D0%B8%D1%97%D0%B2|
| 3 | next.cs-202-02.lab| next.cs-202-02.lab |/orders |
| 4 |www.cs-202-02.lab|www.cs-202-02.lab:4443|/orders/../search |
| 5 |	admin@cs-202-02.lab|- |subject=orders |

Декодоване значення параметра URI 2: id=київ

Чому URI 5 не має складової host, хоча містить символ `@`: У URI 5 використовується схема mailto, яка має іншу структуру, ніж HTTP/HTTPS. У записі admin@cs-202-02.lab символ @ є частиною адреси електронної пошти, а не роздільником між userinfo і host. Тому host у цьому URI відсутній.

### 1.2. Перевірка (після завдання 3)

<!-- Для URI 1–3: рядки запиту (починаються з ">") і тіло відповіді сервера. -->
curl -v http://user:pass@next.cs-202-02.lab/orders
```
* Host next.cs-202-02.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to next.cs-202-02.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /orders HTTP/1.1
> Host: next.cs-202-02.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Thu, 29 Oct 2026 13:19:42 GMT
< Content-Type: text/plain
< Content-Length: 100
< Connection: keep-alive
< 
server: web-b
Host: next.cs-202-02.lab
request URI: /orders
path: /orders
query: 
client: 10.53.1.1
* Connection #0 to host next.cs-202-02.lab left intact
root@ns-cs-202-02:/# 
```
 curl -v 'http://www.cs-202-02.lab/orders/../search'
```
* Host www.cs-202-02.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.cs-202-02.lab (10.53.0.80) port 80
> GET /search HTTP/1.1
> Host: www.cs-202-02.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Thu, 29 Oct 2026 13:26:58 GMT
< Content-Type: text/plain
< Content-Length: 99
< Connection: keep-alive
< 
server: web-a
Host: www.cs-202-02.lab
request URI: /search
path: /search
query: 
client: 10.53.1.1
* Connection #0 to host www.cs-202-02.lab left intact
```
curl -v --path-as-is 'http://www.cs-202-02.lab/orders/../search'
```
* Host www.cs-202-02.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.cs-202-02.lab (10.53.0.80) port 80
> GET /orders/../search HTTP/1.1
> Host: www.cs-202-02.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Thu, 29 Oct 2026 13:27:32 GMT
< Content-Type: text/plain
< Content-Length: 109
< Connection: keep-alive
< 
server: web-a
Host: www.cs-202-02.lab
request URI: /orders/../search
path: /search
query: 
client: 10.53.1.1
* Connection #0 to host www.cs-202-02.lab left intac
```

Значення заголовка `Authorization` і результат декодування:

```
Authorization: Basic dXNlcjpwYXNz

user:pass
```

URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують: DNS-ім'я успішно знайдено (10.53.0.80), але TCP-з'єднання з портом 4443 відхилено (Connection refused). До TLS-з'єднання запит не дійшов.

```
* Host www.cs-202-02.lab:4443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:4443...
* connect to 10.53.0.80 port 4443 from 10.53.1.1 port 36030 failed: Connection refused
* Failed to connect to www.cs-202-02.lab port 4443 after 41 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.cs-202-02.lab port 4443 after 41 ms: Couldn't connect to server
```

| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний |	/search |	/search |
| `--path-as-is` |	/orders/../search | 	/search|

Розбіжності між URI та тим, що отримав сервер, і їх пояснення:

У звичайному запиті curl автоматично змінює шлях /orders/../search на /search. Тому сервер отримує вже скорочений URI /search.
Якщо використати параметр --path-as-is, програма передає початковий шлях без змін — /orders/../search. Проте сервер самостійно обробляє його та визначає кінцевий шлях як /search.
Отже, різниця полягає в тому, що у звичайному запиті шлях нормалізує curl, а з параметром --path-as-is — уже сам сервер
## Завдання 2. Проходження ієрархії вручну

 >dig @10.53.0.2 ns.cs-202-02.lab A +norec 

  ```

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.2 ns.cs-202-02.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1019
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: dc6ce8582c529642010000006ae0dcbba094de55381e77a2 (good)
;; QUESTION SECTION:
;ns.cs-202-02.lab.              IN      A

;; AUTHORITY SECTION:
lab.                    172800  IN      NS      ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.            172800  IN      A       10.53.0.3

;; Query time: 80 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Tue Oct 27 17:17:47 UTC 2026
;; MSG SIZE  rcvd: 117 

 ```
>dig @10.53.0.2 ns.cs-202-02.lab A +norec
 ```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.2 ns.cs-202-02.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1019
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: dc6ce8582c529642010000006ae0dcbba094de55381e77a2 (good)
;; QUESTION SECTION:
;ns.cs-202-02.lab.              IN      A

;; AUTHORITY SECTION:
lab.                    172800  IN      NS      ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.            172800  IN      A       10.53.0.3

;; Query time: 80 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Tue Oct 27 17:17:47 UTC 2026
;; MSG SIZE  rcvd: 117
 ```
> dig @10.53.1.1 ns.cs-202-02.lab A +norec
 ```
 ; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 ns.cs-202-02.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46046
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: c6cf87818afbd4ec010000006ae0dcdc92f8dc8dfc74556d (good)
;; QUESTION SECTION:
;ns.cs-202-02.lab.              IN      A

;; ANSWER SECTION:
ns.cs-202-02.lab.       30      IN      A       10.53.1.1

;; AUTHORITY SECTION:
cs-202-02.lab.          30      IN      NS      ns.cs-202-02.lab.

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Oct 27 17:18:20 UTC 2026
;; MSG SIZE  rcvd: 103
 ```
| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 | NOERROR| qr| 0| lab. 172800 IN NS ns1.nic.lab.|ns1.nic.lab. 172800 IN A 10.53.0.3 |
| 10.53.0.3 |NOERROR |qr |0 | 	cs-202-02.lab. 3600 IN NS ns.cs-202-02.lab.| ns.cs-202-02.lab. 3600 IN A 10.53.1.1|
| 10.53.1.1 | NOERROR| qr aa| ns.cs-202-02.lab. 30 IN A 10.53.1.1|cs-202-02.lab. 30 IN NS ns.cs-202-02.lab. | -|

Вивід `dig ns.cs-202-02.lab A +trace`:

```; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> ns.cs-202-02.lab A +trace
;; global options: +cmd
.                       80632   IN      NS      a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 0 ms

lab.                    172800  IN      NS      ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 79 ms

cs-202-02.lab.          3600    IN      NS      ns.cs-202-02.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 39 ms

ns.cs-202-02.lab.       30      IN      A       10.53.1.1
cs-202-02.lab.          30      IN      NS      ns.cs-202-02.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.cs-202-02.lab) in 0 ms 

```

| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 sNN.lab NS +norec` | 3600|- | AUTHORITY|
| `dig @10.53.1.1 sNN.lab NS +norec` |30 | qr aa| 	ANSWER|

1. Які відповіді є відсиланнями і за якими ознаками це визначено?
```
Відповідь сервера 10.53.0.3 є відсиланням до іншого DNS-сервера. Це означає, що сервер не надав остаточну відповідь на запит, а повідомив, куди потрібно звернутися далі, щоб отримати потрібну інформацію.
Це можна визначити за кількома ознаками у результаті команди dig.
У відповіді відсутній прапорець aa. Цей прапорець означає, що сервер є авторитетним щодо запитуваного доменного імені. Його відсутність показує, що отримана відповідь не є авторитетною. Секція ANSWER порожня (ANSWER: 0). Тобто сервер не повернув IP-адресу, яку безпосередньо запитували.
У секції AUTHORITY міститься запис типу NS, який вказує на DNS-сервер, відповідальний за потрібну зону. У нашому випадку це ns.cs-202-02.lab.
Отже, сервер 10.53.0.3 не надав остаточної відповіді, а передав інформацію про наступний сервер, до якого потрібно звернутися. Саме тому така відповідь називається відсиланням.
```
2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?
```
Секція ADDITIONAL призначена для розміщення додаткової інформації, яка може допомогти під час подальшого проходження DNS-ієрархії.
Ця інформація потрібна для того, щоб резолвер міг одразу визначити, до якого сервера потрібно звернутися для отримання остаточної відповіді. Тобто TLD-сервер не лише повідомляє назву авторитетного сервера через запис NS, але й додатково надає його IP-адресу. Якби запису в секції ADDITIONAL не було, резолвер знав би назву потрібного DNS-сервера, але не мав би його IP-адреси. Тоді довелося б виконати ще один DNS-запит, щоб окремо визначити адресу цього сервера.
```
3. Яка з двох копій запису NS є авторитативною і чому?
```
Авторитативною є копія запису NS, отримана безпосередньо від сервера 10.53.1.1. Це пояснюється тим, що під час звернення до цього сервера у відповіді присутній прапорець aa. Він підтверджує, що сервер має авторитетну інформацію про відповідну DNS-зону.У відповіді сервера 10.53.0.3 запис NS знаходиться в секції AUTHORITY, але прапорець aa відсутній. Такий запис використовується для відсилання резолвера до потрібного сервера. Тобто обидві копії містять інформацію про DNS-сервер зони, але мають різне призначення. Копія від TLD-сервера допомагає знайти потрібний сервер, а копія від власного авторитетного сервера є частиною його власної авторитетної інформації.
```

## Завдання 3. Заповнення власної зони

Вивід `zone-reload`:

```
zone cs-202-02.lab/IN: loaded serial 2026102901
OK
zone reload up-to-date

```

 dig @10.53.1.1 www.cs-202-02.lab A +norec
```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 www.cs-202-02.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43629
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 2725930afb56622f010000006ae341550f173771c0093b6f (good)
;; QUESTION SECTION:
;www.cs-202-02.lab.             IN      A

;; ANSWER SECTION:
www.cs-202-02.lab.      30      IN      A       10.53.0.80

;; AUTHORITY SECTION:
cs-202-02.lab.          30      IN      NS      ns.cs-202-02.lab.

;; ADDITIONAL SECTION:
ns.cs-202-02.lab.       30      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Thu Oct 29 12:51:33 U
```
dig @10.53.1.1 portal.cs-202-02.lab CNAME +norec
```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 portal.cs-202-02.lab CNAME +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47156
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b7e56318e25b03de010000006ae3419a60223b9b64bb9859 (good)
;; QUESTION SECTION:
;portal.cs-202-02.lab.          IN      CNAME

;; ANSWER SECTION:
portal.cs-202-02.lab.   30      IN      CNAME   www.cs-202-02.lab.

;; AUTHORITY SECTION:
cs-202-02.lab.          30      IN      NS      ns.cs-202-02.lab.

;; ADDITIONAL SECTION:
ns.cs-202-02.lab.       30      IN      A       10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Thu Oct 29 12:52:42 UTC 2026
;; MSG SIZE  rcvd: 128
```
dig @10.53.1.1 mail.cs-202-02.lab A +norec
 ```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 mail.cs-202-02.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32076
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 85725c759c988b1e010000006ae341d9d033209b6ef78392 (good)
;; QUESTION SECTION:
;mail.cs-202-02.lab.            IN      A

;; ANSWER SECTION:
mail.cs-202-02.lab.     30      IN      A       10.53.0.80

;; AUTHORITY SECTION:
cs-202-02.lab.          30      IN      NS      ns.cs-202-02.lab.

;; ADDITIONAL SECTION:
ns.cs-202-02.lab.       30      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Thu Oct 29 12:53:45 UTC 2026
;; MSG SIZE  rcvd: 124
 ```
dig @10.53.1.1 cs-202-02.lab MX +norec
 ```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 cs-202-02.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10861
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: bfe0f26022e674f5010000006ae341f96cfe319a87054ee9 (good)
;; QUESTION SECTION:
;cs-202-02.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-02.lab.          30      IN      MX      15 mail.cs-202-02.lab.

;; AUTHORITY SECTION:
cs-202-02.lab.          30      IN      NS      ns.cs-202-02.lab.

;; ADDITIONAL SECTION:
mail.cs-202-02.lab.     30      IN      A       10.53.0.80
ns.cs-202-02.lab.       30      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Thu Oct 29 12:54:17 UTC 2026
;; MSG SIZE  rcvd: 140
 ```
dig @10.53.1.1 cs-202-02.lab TXT +norec
 ```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 cs-202-02.lab TXT +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39938
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 34063f1f9f595a4b010000006ae3428a54d70ffbcf48a6b6 (good)
;; QUESTION SECTION:
;cs-202-02.lab.                 IN      TXT

;; ANSWER SECTION:
cs-202-02.lab.          30      IN      TXT     "lab-token=f71ca29"

;; AUTHORITY SECTION:
cs-202-02.lab.          30      IN      NS      ns.cs-202-02.lab.

;; ADDITIONAL SECTION:
ns.cs-202-02.lab.       30      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Thu Oct 29 12:56:42 UTC 2026
;; MSG SIZE  rcvd: 133
 ```
dig @10.53.1.1 next.cs-202-02.lab A +norec
 ```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 next.cs-202-02.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21185
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 1c054ea9f570143f010000006ae342b42137e674278f7289 (good)
;; QUESTION SECTION:
;next.cs-202-02.lab.            IN      A

;; ANSWER SECTION:
next.cs-202-02.lab.     20      IN      A       10.53.0.81

;; AUTHORITY SECTION:
cs-202-02.lab.          30      IN      NS      ns.cs-202-02.lab.

;; ADDITIONAL SECTION:
ns.cs-202-02.lab.       30      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Thu Oct 29 12:57:24 UTC 2026
;; MSG SIZE  rcvd: 124

 ```
| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A | NOERROR| є|	10.53.0.80 |30 | так|
| псевдонім CNAME |NOERROR |є |www.cs-202-02.lab. | 30| так|
| `mail` A |NOERROR |є | 10.53.0.80| 30|так |
| `@` MX |NOERROR|є |15 mail.cs-202-02.lab. |30 |так |
| `@` TXT |NOERROR |є | "lab-token=f71fca29"| 30| так|
| другий хост A | NOERROR|є | 10.53.0.81|20 |так |

Порівняння відповідей резолвера та власного сервера:

```
dig portal.cs-202-02.lab

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> portal.cs-202-02.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7574
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;portal.cs-202-02.lab.          IN      A

;; ANSWER SECTION:
portal.cs-202-02.lab.   30      IN      CNAME   www.cs-202-02.lab.
www.cs-202-02.lab.      30      IN      A       10.53.0.80

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Thu Oct 29 13:08:25 UTC 2026
;; MSG SIZE  rcvd: 83
```
 dig @10.53.1.1 portal.cs-202-02.lab +norec
 ```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 portal.cs-202-02.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18325
;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 19c3d11ce46e376d010000006ae34573bd550fa088cfbeec (good)
;; QUESTION SECTION:
;portal.cs-202-02.lab.          IN      A

;; ANSWER SECTION:
portal.cs-202-02.lab.   30      IN      CNAME   www.cs-202-02.lab.
www.cs-202-02.lab.      30      IN      A       10.53.0.80

;; AUTHORITY SECTION:
cs-202-02.lab.          30      IN      NS      ns.cs-202-02.lab.

;; ADDITIONAL SECTION:
ns.cs-202-02.lab.       30      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Thu Oct 29 13:09:07 UTC 2026
;; MSG SIZE  rcvd: 144
 ```
 dig cs-202-02.lab MX
 ```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> cs-202-02.lab MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36269
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;cs-202-02.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-02.lab.          30      IN      MX      15 mail.cs-202-02.lab.

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Thu Oct 29 13:09:30 UTC 2026
;; MSG SIZE  rcvd: 63
 ```
dig @10.53.1.1 cs-202-02.lab MX +norec
 ```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 cs-202-02.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10224
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 1c201dfa15b2ca10010000006ae345a234287a50e9e01228 (good)
;; QUESTION SECTION:
;cs-202-02.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-02.lab.          30      IN      MX      15 mail.cs-202-02.lab.

;; AUTHORITY SECTION:
cs-202-02.lab.          30      IN      NS      ns.cs-202-02.lab.

;; ADDITIONAL SECTION:
mail.cs-202-02.lab.     30      IN      A       10.53.0.80
ns.cs-202-02.lab.       30      IN      A       10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Thu Oct 29 13:09:54 UTC 2026
;; MSG SIZE  rcvd: 140
 ```
| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a |qr rd ra | 2, CNAME + A| 0|1, OPT* |30 |
| псевдонім через 10.53.1.1 | qr aa|2, CNAME + A |	1, NS |	2, A + OPT* |30 |
| MX через resolver-a |qr rd ra | 1, MX|0 |1, OPT* |30 |
| MX через 10.53.1.1 |qr aa | 1, MX| 1, NS|	3, A + A + OPT* |30 |

Пояснення відмінностей:

Під час порівняння відповідей було встановлено, що основні DNS-записи в обох випадках однакові, але відрізняється структура відповідей.
Через resolver-a відповідь має прапорці rd ra, тобто є рекурсивною. Прапорець aa відсутній, а секція AUTHORITY порожня.
Під час виконання запиту типу A для portal.cs-202-02.lab було отримано CNAME-запис, який вказує на інше доменне ім’я, а також A-запис із IP-адресою 10.53.0.80. Це показує, що DNS спочатку визначає цільове ім’я через CNAME, а потім знаходить його IP-адресу.
Для MX-запиту також отримано однаковий основний результат, але авторитетний сервер додатково повертає NS та A-записи в інших секціях.
Отже, основна різниця полягає у способі отримання відповіді та наявності додаткової авторитетної інформації.


Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:

```
dns_master_load: /lab/zone/cs-202-02.lab.zone:21: cs-202-02.lab: CNAME and other data
zone cs-202-02.lab/IN: loading from master file /lab/zone/cs-202-02.lab.zone failed: CNAME and other data
zone cs-202-02.lab/IN: not loaded due to errors.
```

**1. Який TTL має кожен запис зони і звідки це значення взято?**

У створеній DNS-зоні було задано загальне значення $TTL 30. Це означає, що для записів, у яких не вказано власне значення TTL, автоматично використовується час життя 30 секунд.Тому записи www, portal, mail, MX та TXT мають TTL 30 секунд. Окремо було налаштовано запис next, для якого TTL зазначено безпосередньо в самому записі. Його значення становить 20 секунд. Для SOA-запису у файлі зони окремо задано TTL 120 секунд. Також у параметрах SOA присутнє значення minimum = 60. Воно використовується для визначення часу негативного кешування, а не як загальний TTL для всіх записів зони. Таким чином, значення TTL залежить від налаштувань конкретного запису та загального параметра $TTL. Якщо окреме значення не задане, використовується загальне значення 30 секунд.

**2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?**

Під час запиту до portal.cs-202-02.lab у секції ANSWER було отримано два записи. Перший запис має тип CNAME. Він показує, що portal.cs-202-02.lab є псевдонімом і посилається на www.cs-202-02.lab. Другий запис має тип A та містить IP-адресу 10.53.0.80, яка належить імені www.cs-202-02.lab. Таким чином, DNS повертає не лише інформацію про сам псевдонім, а й адресу доменного імені, на яке він посилається. Завдяки цьому клієнт може отримати кінцеву IP-адресу, не виконуючи окремий запит для кожного переходу. Отже, секція ANSWER містить два записи: один CNAME та один A.

**3. Чому запис CNAME неможливо розмістити для імені `@`?**

Ім'я `@` відповідає вершині зони `cs-202-02.lab`, для якої вже існують SOA, NS, MX та TXT записи. CNAME не може співіснувати з іншими типами записів для одного імені. Тому додавання `@ IN CNAME www` викликає помилку `CNAME and other data` під час перевірки зони командою `zone-check`. Отже, CNAME не можна використовувати для `@`, оскільки для вершини зони вже визначені інші записи.

## Завдання 4. TTL і видимість змін

Два запити з інтервалом 5 с:

```
www.cs-202-02.lab.  30  IN  A  10.53.0.80

www.cs-202-02.lab.  22  IN  A  10.53.0.80

```

**Прогноз** (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка:

`Tпояви ≈ Tзміни + R`

де `R` — залишок TTL на момент зміни
Для `$TTL = 30 с` максимальна затримка — 30 с


Перевірка, що сервер до `zone-reload` повертає стару адресу:

```
root@ns-cs-202-02:/# dig @10.53.1.1 www.cs-202-02.lab +norec +short
10.53.0.80
```
```
root@ns-cs-202-02:/# date +%T; dig www.cs-202-02.lab +noall +answer
13:50:14
www.cs-202-02.lab.      30      IN      A       10.53.0.80
```
 
```
root@ns-cs-202-02:/# zone-reload
date +%T
zone cs-202-02.lab/IN: loaded serial 2026102903
OK
zone reload queued
13:52:04
```
```
root@ns-cs-202-02:/#  dig @10.53.1.1 www.cs-202-02.lab +norec +short
10.53.0.81
```


**Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):**


На кроці 4 залишок TTL становив `R = 30 с`. О `13:52:04` виконано `zone-reload`, після чого авторитетний сервер почав повертати нову адресу `10.53.0.81`.

```
Очікуваний час появи нової адреси:

13:52:04 + 30 с = 13:52:34
```

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):
```
root@ns-cs-202-02:/# while true; do echo "$(date +%T) $(dig +short www.cs-202-02.lab)"; sleep 2; done
14:02:49 10.53.0.81
14:02:51 10.53.0.81
14:02:53 10.53.0.81
14:02:55 10.53.0.81
14:02:57 10.53.0.81
14:02:59 10.53.0.81
14:03:01 10.53.0.81
14:03:03 10.53.0.81
14:03:05 10.53.0.81
^C
```


Рядок `server:` з відповіді `curl`:
```
server: web-b
```

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| 13:52:04|13:52:34 |	14:02:49 |- |30 |


**1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?**

Прогноз передбачав, що після зміни IP-адреси резолвер ще певний час повертатиме старе значення через наявність запису в кеші.
Однак під час фактичного спостереження вже було отримано нову адресу 10.53.0.81. Тобто очікуваного періоду, протягом якого можна було б зафіксувати стару адресу 10.53.0.80, у наведеному циклі не спостерігалося.
Це можна пояснити тим, що цикл перевірки було запущено вже після виконання zone-reload. Крім того, DNS-запити виконувалися з певним інтервалом, тому точний момент переходу від старої адреси до нової не був зафіксований.
У результаті ми бачимо лише те, що під час першої наведеної перевірки резолвер уже повертав нову адресу.
Тому точну фактичну тривалість кешування старого запису за цими вимірюваннями визначити неможливо. Для цього потрібно було б спостерігати за відповідями безпосередньо до моменту зміни адреси та одразу після неї.

**2. Якою була б найбільша можлива затримка для цього запису і за яких умов?**

Для цього запису TTL становить **30 секунд**, тому найбільша можлива затримка поширення нової адреси через DNS-кеш становила б приблизно **30 секунд**. Така ситуація можлива, якщо старе значення запису було отримано та закешовано безпосередньо перед зміною IP-адреси. У такому випадку DNS-резолвер продовжуватиме використовувати стару адресу до завершення TTL, після чого звернеться за актуальним значенням.

**3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?**

TTL **86 400 секунд** дорівнює 24 годинам, тому після зміни IP-адреси деякі DNS-резолвери можуть використовувати стару адресу ще протягом доби. Щоб правильно перенести сервіс, потрібно заздалегідь зменшити TTL запису, наприклад до невеликого значення, і почекати щонайменше один попередній період TTL, щоб старі записи в кешах DNS-серверів поступово зникли. Після цього можна змінити IP-адресу запису та виконати оновлення DNS-зони. Після перенесення сервісу бажано переконатися, що нова адреса вже використовується, і лише після цього повернути TTL до звичайного значення.

## Завдання 5. Негативне кешування

**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення: 
Перед виконанням експерименту було визначено, що під час запиту до неіснуючого доменного імені DNS-сервер повертає відповідь NXDOMAIN Ця відповідь означає, що запитуваного імені не існує в DNS-зоні. Особливість негативного кешування полягає в тому, що резолвер може зберігати не лише успішні відповіді, але й інформацію про те, що певне ім'я не існує.
TTL визначається за формулою:
TTL негативного кешування = min(120 с, 60 с) = 60 с

Тобто для негативного кешування використовується менше зі значень TTL SOA та параметра minimum.
Отже, прогнозована тривалість негативного кешування становить 60 секунд.
Передбачалося, що якщо спочатку виконати запит до неіснуючого імені, а потім додати його до DNS-зони, резолвер не обов'язково одразу почне повертати нову адресу. Він може продовжувати використовувати раніше збережену негативну відповідь NXDOMAIN до завершення її TTL.

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:

```
root@ns-cs-202-02:/# dig @10.53.1.1 new-zfkmu.cs-202-02.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new-zfkmu.cs-202-02.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 58396
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: be755f4d6dea1e12010000006ae355cc9bc0be47267173fb (good)
;; QUESTION SECTION:
;new-zfkmu.cs-202-02.lab.       IN      A

;; AUTHORITY SECTION:
cs-202-02.lab.          60      IN      SOA     ns.cs-202-02.lab. admin.cs-202-02.lab. 2026102903 3600 600 86400 60

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Thu Oct 29 14:18:52 UTC 2026
;; MSG SIZE  rcvd: 125
```

Запит через resolver-a (крок 3), час zone-reload (крок 4), перевірка на сервері (крок 5):

date +%T; dig new-zfkmu.cs-202-02.lab
```
14:23:24

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> new-zfkmu.cs-202-02.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 48075
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new-zfkmu.cs-202-02.lab.       IN      A

;; AUTHORITY SECTION:
cs-202-02.lab.          60      IN      SOA     ns.cs-202-02.lab. admin.cs-202-02.lab. 2026102903 3600 600 86400 60

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Thu Oct 29 14:23:24 UTC 2026
;; MSG SIZE  rcvd: 97
```
zone-reload & date +%T
```
zone cs-202-02.lab/IN: loaded serial 2026102904
OK
zone reload up-to-date
```
dig @10.53.1.1 new-zfkmu.cs-202-02.lab +norec +short
```
10.53.0.80
```
Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):

```
14:34:44 status: NOERROR
14:34:46 status: NOERROR
14:34:48 status: NOERROR
14:34:50 status: NOERROR
```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| 14:23:24|	14:31:34 |	14:34:44 | 	680|60 |


1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?
   
Між першим негативним запитом о 14:23:24 та отриманням успішної відповіді о 14:34:44 минуло 680 секунд. Однак цей проміжок не можна вважати точним часом негативного кешування, оскільки запис було створено лише о 14:31:34. Тому за отриманими результатами неможливо точно визначити тривалість зберігання негативної відповіді в кеші.

2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.

У нашому експерименті TTL SOA у відповіді NXDOMAIN становив 60 секунд. Таке значення пов'язане з параметром minimum = 60, який задано у файлі DNS-зони. Водночас у самому файлі зони для SOA окремо зазначено TTL 120 секунд. Тому значення 60 секунд у відповіді NXDOMAIN не означає, що TTL SOA у файлі зони також дорівнює 60 секундам.

3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?

До створення DNS-запису скрипт отримає відповідь NXDOMAIN, оскільки потрібного імені ще немає в зоні. Після цього, якщо скрипт одразу створить запис і повторить запит через той самий резолвер, він може знову отримати NXDOMAIN. Причина полягає в негативному кешуванні. Резолвер уже зберіг попередню інформацію про те, що ім'я не існує, тому може продовжувати повертати цю відповідь навіть після появи нового запису на авторитетному сервері. Щоб уникнути такої ситуації, потрібно враховувати час негативного кешування. У нашому випадку прогнозований TTL становить 60 секунд, тому після створення запису можна дочекатися завершення цього періоду та повторити перевірку. Також для перевірки можна звернутися безпосередньо до авторитетного DNS-сервера, щоб переконатися, що новий запис уже доданий до зони та правильно завантажений.

## Завдання 6. Затримка пошуку імені

**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні | 40 мс|1 мс |
| B: нове ім'я в зоні `lab.` | 40 мс|0 мс |
| C: ім'я в неіснуючому TLD |80 мс |0 мс |
| D: `www.sNN.lab` після перезапуску резолвера |401  мс |0 мс |

 A=y$RANDOM.sNN.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'

```
;; Query time: 40 msec
;; Query time: 1 msec
```
B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
```
;; Query time: 40 msec
;; Query time: 0 msec
```
C=x$RANDOM.zz$RANDOM; dig $C | grep 'Query time'; dig $C | grep 'Query time'
```
;; Query time: 80 msec
;; Query time: 0 msec
```

dig www.cs-202-02.lab | grep 'Query time'; dig www.cs-202-02.lab | grep 'Query time'
```
;; Query time: 401 msec
;; Query time: 0 msec
```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A | 40| 1 |власний (querylog: y17378.cs-202-02.lab) |
| B | 40| 0|TLD (x15082.lab)|
| C | 80| 0|root (x23919.zz22475) |
| D | 401|0 | root-TLD-власний (www.cs-202-02.lab) |

Значення `time_namelookup` з `curl` після перезапуску резолвера:0.404003 с

Фрагменти журналів запитів root, TLD і власного сервера для рядка D:

root
```
29-Oct-2026 12:35:51.739 client @0x70cb3006ac88 10.53.1.1#33553 (ns.sNN.lab): query: ns.sNN.lab IN A -E(0)K (10.53.0.2)
29-Oct-2026 12:36:01.199 client @0x70cb38004488 10.53.1.1#45358 (ns.cs-202-02.lab): query: ns.cs-202-02.lab IN A -E(0)K (10.53.0.2)
29-Oct-2026 14:47:59.806 client @0x70cb38004488 10.53.0.53#40451 (x23919.zz22475): query: x23919.zz22475 IN A -E(0)DC (10.53.0.2)
29-Oct-2026 14:52:41.802 client @0x70cb38004488 10.53.0.53#13533 (.): query: . IN NS -E(0)DC (10.53.0.2)
29-Oct-2026 14:52:41.882 client @0x70cb3006ac88 10.53.0.53#16629 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
29-Oct-2026 14:52:41.963 client @0x70cb3006ac88 10.53.0.53#49022 (www.cs-202-02.lab): query: www.cs-202-02.lab IN A -E(0)DC (10.53.0.2)
29-Oct-2026 14:55:15.899 client @0x70cb3006ac88 10.53.0.53#17961 (.): query: . IN NS -E(0)DC (10.53.0.2)
29-Oct-2026 14:55:15.980 client @0x70cb3006ac88 10.53.0.53#13595 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
29-Oct-2026 14:55:16.060 client @0x70cb38004488 10.53.0.53#46695 (www.cs-202-02.lab): query: www.cs-202-02.lab IN A -E(0)DC (10.53.0.2)
29-Oct-2026 14:55:16.060 client @0x70cb38006428 10.53.0.53#36888 (www.cs-202-02.lab): query: www.cs-202-02.lab IN AAAA -E(0)DC (10.53.0.2)
```
TLD:
```
29-Oct-2026 13:00:49.542 client @0x7e7a6806ac88 10.53.0.53#18026 (portal.cs-202-02.lab): query: portal.cs-202-02.lab IN A -E(0)DC (10.53.0.3)
29-Oct-2026 13:33:49.269 client @0x7e7a6806ac88 10.53.0.53#28094 (www.cs-202-02.lab): query: www.cs-202-02.lab IN A -E(0)DC (10.53.0.3)
29-Oct-2026 13:33:49.269 client @0x7e7a69293cc8 10.53.0.53#42484 (www.cs-202-02.lab): query: www.cs-202-02.lab IN AAAA -E(0)DC (10.53.0.3)
29-Oct-2026 14:02:49.357 client @0x7e7a69293cc8 10.53.0.53#38778 (www.cs-202-02.lab): query: www.cs-202-02.lab IN A -E(0)DC (10.53.0.3)
29-Oct-2026 14:34:44.138 client @0x7e7a69293cc8 10.53.0.53#28226 (new-zfkmu.cs-202-02.lab): query: new-zfkmu.cs-202-02.lab IN A -E(0)DC (10.53.0.3)
29-Oct-2026 14:47:03.899 client @0x7e7a60005268 10.53.0.53#35654 (y16846.sNN.lab): query: y16846.sNN.lab IN A -E(0)DC (10.53.0.3)
29-Oct-2026 14:47:37.259 client @0x7e7a60005268 10.53.0.53#11696 (x15082.lab): query: x15082.lab IN A -E(0)DC (10.53.0.3)
29-Oct-2026 14:52:42.083 client @0x7e7a60005268 10.53.0.53#55559 (www.cs-202-02.lab): query: www.cs-202-02.lab IN A -E(0)DC (10.53.0.3)
29-Oct-2026 14:55:16.181 client @0x7e7a60005268 10.53.0.53#21020 (www.cs-202-02.lab): query: www.cs-202-02.lab IN A -E(0)DC (10.53.0.3)
29-Oct-2026 14:55:16.181 client @0x7e7a69293cc8 10.53.0.53#21153 (www.cs-202-02.lab): query: www.cs-202-02.l
```
1. У цьому стенді за моїми вимірюваннями приблизні додаткові затримки становлять: root ≈80 мс (`C`), TLD ≈40 мс (`B`), власного авторитативного сервера ≈1 мс (`A`). Це оцінки за сумарним часом DNS-запиту, а не ізольовані значення чистого RTT. Для рядка `D` перший запит становив 401 мс, оскільки під час пошуку імені resolver-a звертався послідовно до root `10.53.0.2`, TLD `10.53.0.3` та власного сервера `10.53.1.1`.

2. Для запиту A перше виконання зайняло 40 мс, а повторне — лише 1 мс. Для запиту B час першого виконання становив 40 мс, для C — 80 мс, а для D — 401 мс. Під час повторних запитів час значно зменшився, оскільки відповіді вже зберігалися в кеші DNS-резолвера. Найбільшу затримку спостерігали для запиту D після перезапуску резолвера.
   
3.Для рядків B і C перший запит вимагав пошуку через DNS-ієрархію, тому він тривав довше: 40 мс для B та 80 мс для C. Після отримання відповіді резолвер зберіг її у своєму кеші. Через це під час повторного запиту йому вже не потрібно було знову звертатися до інших DNS-серверів, тому час зменшився до 0 мс. У рядку D після перезапуску resolver-a попередній кеш уже не використовувався. Тому перший запит знову вимагав звернення до Root, TLD та власного авторитативного сервера, через що його тривалість становила 401 мс. Повторний запит виконувався значно швидше — за 0 мс, оскільки результат уже був збережений у кеші.

## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):

```
root@ns-cs-202-02:/# dig @10.53.0.53 next.cs-202-02.lab +noall +answer
next.cs-202-02.lab.     20      IN      A       10.53.0.81

root@ns-cs-202-02:/# dig @10.53.0.54 next.cs-202-02.lab +noall +answer
next.cs-202-02.lab.     20      IN      A       10.53.0.81
```

Запити із заміром часу (крок 5):

time dig @10.53.0.53 next.cs-202-02.lab +tries=1 +timeout=30
```
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 next.cs-202-02.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real    0m30.048s
user    0m0.005s
sys     0m0.007s
```

time dig @10.53.0.53 next.cs-202-02.lab +tries=1 +timeout=30
```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 next.cs-202-02.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 2959
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;next.cs-202-02.lab.            IN      A

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Thu Oct 29 15:37:19 UTC 2026
;; MSG SIZE  rcvd: 47


real    0m0.011s
user    0m0.005s
sys     0m0.006s
```
root@ns-cs-202-02:/# time dig @10.53.0.54 next.cs-202-02.lab +tries=1 +timeout=30
```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.54 next.cs-202-02.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51982
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;next.cs-202-02.lab.            IN      A

;; ANSWER SECTION:
next.cs-202-02.lab.     30      IN      A       10.53.0.81

;; Query time: 1800 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Thu Oct 29 15:37:42 UTC 2026
;; MSG SIZE  rcvd: 63


real    0m1.822s
user    0m0.004s
sys     0m0.007s
```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший |Тайм-аут, no servers could be reached|- | 	0m30.048s|
| resolver-a, другий | 	SERVFAIL|- |0m0.011s |
| resolver-b | 	NOERROR|next.cs-202-02.lab. 30 IN A 10.53.0.81 |0m1.822s |

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```
15:38:50 10.53.0.81
15:38:52 10.53.0.81
15:38:54 10.53.0.81

```

Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):

```
ns.cs-202-02.lab. admin.cs-202-02.lab. 2026102904 3600 600 86400 60
```

### Постмортем

**1. Що сталося:**  
У відповіді resolver-b запис мав TTL 30 секунд:
next.cs-202-02.lab.  30  IN  A  10.53.0.81
Це застарілий запис, який резолвер повернув завдяки механізму serve-stale. Його призначення — забезпечити доступність DNS-відповіді навіть тоді, коли авторитативний сервер тимчасово не працює. Завдяки цьому клієнти можуть продовжувати отримувати раніше відому IP-адресу, а не одразу стикатися з помилкою.

**2. Вплив на клієнтів resolver-a і resolver-b:**  
Результати двох запитів відрізнялися. Перший запит завершився тайм-аутом через 30.048 секунди. Це означає, що resolver-a очікував відповідь від авторитативного сервера, але не зміг її отримати, оскільки сервер був зупинений. Другий запит завершився значно швидше — за 0.011 секунди зі статусом SERVFAIL. Можна припустити, що після першої невдалої спроби резолвер уже визначив, що сервер недоступний, тому не став повторно чекати протягом 30 секунд. Водночас точний внутрішній механізм такої зміни поведінки за отриманими журналами визначити не вдалося.

**3. Хронологія з виміряним часом:**  
Після запуску авторитативного сервера resolver-a не обов'язково одразу отримав актуальну відповідь. Йому потрібно було знову звернутися до DNS-сервера та отримати інформацію про потрібний запис. Також на результат могли впливати попередні невдалі спроби запитів. За результатами спостережень перша успішна відповідь з IP-адресою 10.53.0.81 була отримана о 15:38:50. Після цього наступні перевірки також повертали цю адресу.

**4. Чому кеш допоміг або не допоміг:**  
Кеш допоміг `resolver-b`, тому що механізм `serve-stale` дозволив використати застарілий запис після завершення TTL. Для `resolver-a` звичайний кеш уже не міг використовуватися після спливання TTL, тому він не зміг повернути актуальну відповідь і звертався до недоступного авторитативного сервера.

**5. Що слід змінити:**  
Для підвищення надійності потрібно налаштувати резервний авторитативний DNS-сервер, щоб у разі зупинки основного залишалася можливість отримувати актуальні відповіді. Також потрібно правильно налаштовувати TTL і враховувати особливості serve-stale, оскільки він може повертати застарілі записи

**1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?**

У відповіді `resolver-b` запис мав TTL **30 секунд**:

```text
next.cs-202-02.lab.  30  IN  A  10.53.0.81
```

Це застарілий запис, який `resolver-b` повернув завдяки механізму **serve-stale**. Такий механізм потрібен для того, щоб під час недоступності авторитативного DNS-сервера клієнти все одно могли отримати вже відому адресу з кешу.

**2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.**

Результати запитів суттєво відрізнялися. Перший запит завершився тайм-аутом через **30.048 с**, оскільки `resolver-a` намагався отримати відповідь від недоступного авторитативного сервера. Другий запит завершився значно швидше — за **0.011 с** зі статусом `SERVFAIL`. Ймовірно, резолвер уже визначив, що авторитативний сервер недоступний, тому повторно не витрачав 30 секунд на очікування відповіді.

**3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?**

Після запуску авторитативного сервера resolver-a не одразу почав повертати успішну відповідь. Йому потрібно було знову звернутися до DNS-сервера та отримати актуальну інформацію про потрібний запис. За результатами спостережень перша успішна відповідь з IP-адресою 10.53.0.81 була отримана о 15:38:50. Після цього наступні перевірки також повертали цю адресу. Точний час між запуском сервера та першою успішною відповіддю визначити неможливо, оскільки час запуску в результатах не зафіксовано.

**4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.**

`serve-stale` може шкодити, коли DNS-запис уже змінився, але авторитативний сервер недоступний, тому резолвер продовжує повертати стару адресу з кешу. У завданні 4 адресу `www` було змінено з `10.53.0.80` на `10.53.0.81`. У такій ситуації `serve-stale` може продовжити направляти клієнтів на стару адресу. Тобто `serve-stale` підвищує доступність, але може призвести до використання застарілих DNS-даних.

## Фінальний файл зони

<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->

```
; Zone cs-202-02.lab  ^`^t starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 30
@     120 IN SOA ns.cs-202-02.lab. admin.cs-202-02.lab. (
            2026102904 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            60         ; minimum
        )
@     IN NS ns.cs-202-02.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----
www IN A 10.53.0.81
portal IN CNAME www
mail IN A 10.53.0.80
@ IN MX 15 mail
@ IN TXT "lab-token=f71ca29"
new-zfkmu IN A 10.53.0.80
next 20 IN A 10.53.0.81


```

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:

 Очікувалося, що після зміни A-запису з `10.53.0.80` на `10.53.0.81` резолвер ще певний час повертатиме стару адресу через кеш відповідно до TTL. Однак під час першої перевірки циклу спостереження вже була отримана нова адреса:

```text
14:02:49 10.53.0.81
```

Тобто очікуваної затримки з видачею старої адреси `10.53.0.80` у момент спостереження не було зафіксовано.


**Яке питання залишилося без відповіді.** 
**1. (Завдання 4).** Не вдалося точно пояснити, чому нова адреса `10.53.0.81` з'явилася вже під час першої перевірки, хоча очікувалося кешування старої `10.53.0.80` протягом TTL 30 с. Це видно з рядка `14:02:49 10.53.0.81`. Було перевірено TTL, `zone-reload` і авторитетний сервер, але точний момент очищення кешу не зафіксовано.

**2. (Завдання 5).** Не вдалося точно виміряти час негативного кешування `NXDOMAIN`. Спочатку сервер повертав `NXDOMAIN`, але на початку циклу вже отримано `14:34:44 status: NOERROR`. SOA із `minimum = 60 с` було перевірено, але момент завершення негативного кешування не вдалося зафіксувати.

**3. (Завдання 7).** Не вдалося повністю пояснити, чому перший запит до `resolver-a` завершився тайм-аутом за `30.048 с`, а другий — майже одразу з `SERVFAIL` за `0.011 с`. Було перевірено, що авторитетний сервер зупинений і TTL запису вже сплив. Точний внутрішній механізм зміни поведінки `resolver-a` за отриманими журналами визначити не вдалося.

## Декларація використання ШІ

Використовувалися інструменти ШІ: так

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | GPT-6|Допоможи сформулювати коротку відповідь на це питання на основі результатів лабораторної роботи. |Допоміг сформулювати короткі та зрозумілі відповіді на контрольні питання на основі отриманих результатів. |
|2|GPT-6|Проаналізуй цей вивід і скажи, що саме потрібно написати у відповідь на контрольне питання.|Допоміг проаналізувати вивід команд та визначити, які дані потрібно використати у відповіді|
|3|GPT-6|Я виконала це завдання таким способом. Чи правильно я зробила і чи підходить цей результат для звіту?|Допоміг перевірити отриманий результат і зрозуміти, чи відповідає він вимогам конкретного завдання.|
|4|GPT-6|У методичці потрібно отримати ці дані, але я не розумію, якою командою це зробити. Підкажи конкретну команду та скажи, звідки вона взята.|Допоміг розібратися з виконанням окремих етапів лабораторної та відрізнити команди з методички від додаткових команд.|
