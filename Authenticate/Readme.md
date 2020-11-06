# Authenticate 🛅

Security, Owasp top 10, Cron. (#nmap, #lfi, #gobuster, #session #javascript, #curl, #ssh2python.py, #linpeas.sh, #upload_file_nc.sh )

## Tabla de contenido

- [Enumeración](#Enumeración).
- [Web Site](#Web-Site).
- [Shell](#Shell). 


--------------------------------
### Enumeración
-------------------------------
Hacemos una revisión general:
```plain
nmap -sC -sV -oN nmap/initial 10.10.71.122
```

* -oN: Guardar el sondeo en formato normal.
* -sC: Realizar análisis con los scripts por defecto.
* -sV: Detección de la versión de servicios.

Resultado:
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 11:8e:34:34:df:42:17:7b:ab:dc:44:9b:03:17:36:bb (RSA)
|   256 a5:d7:ed:38:db:04:e3:ef:8c:8a:14:25:06:94:f5:a9 (ECDSA)
|_  256 f4:f0:ea:5d:64:b2:3e:cf:16:7d:ef:a8:67:a3:43:0c (ED25519)
5000/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
|_http-title: Live demonstrations
7777/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
|_http-title: No Auth
8888/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
|_http-title: Auth hacks
MAC Address: 02:CC:F0:75:A4:C9 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
--------------------------------
### Dictionary Attack
-------------------------------
El ataque utilizado se hace a través del diccionario rockyou.txt, agregado a través del burpsuite para los siuientes usuarios:
* Jack
* Mike

--------------------------------
### Re-registration
-------------------------------
Utilizamos el endpoint de registro de usuario registrando " arthur", el cuál ya existe, el nuevo registro permite acceder al usuario existente.

--------------------------------
### JSON Web Token
-------------------------------

