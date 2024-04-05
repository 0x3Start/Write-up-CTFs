<h1 align="center"> TheHackersLabs: <a href="https://thehackerslabs.com/mortadela/">Moratedela</a></h1/>

## NetDiscover y NMAP

En este caso voy a utilizar la herramienta netdiscover para detectar la ip de la maquina
```bash
┌──(kali㉿kali)-[~]
└─$ sudo netdiscover -r 10.0.0.0/24

 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                       
                                                                                                                                                                     
 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240                                                                                                     
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 10.0.0.1        52:54:00:12:35:00      1      60  Unknown vendor                                                                                                                                                                                            
 10.0.0.156      08:00:27:ee:58:5c      1      60  PCS Systemtechnik GmbH                                                                                            
```
-r : elegir el rango de de la red   
En este caso al ser una maquina virtual los primero 24 bits siempre son "08:00:27:XX:XX:XX"

Ahora vamos a hacer un escaneo rapido para saber que puertos se encuentran abiertos
```bash
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p- -sS --min-rate 4000 -Pn -n 10.0.0.156
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-05 12:05 EDT
Nmap scan report for 10.0.0.156
Host is up (0.00015s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
MAC Address: 08:00:27:EE:58:5C (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 3.08 seconds
```
 
Una vez que sabemos que puertos tiene abierto vamos a ejecutar un escaneo mas centrado en detectar la version y ejecutar algunos scripts principales
 
 
```bash

┌──(kali㉿kali)-[~]
└─$ nmap -p22,80,3306 -sCV 10.0.0.156
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-05 12:09 EDT
Nmap scan report for 10.0.0.156
Host is up (0.00042s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 aa:8d:e4:75:bc:f3:f8:5e:42:d0:ee:ca:e2:c4:0b:97 (ECDSA)
|_  256 ae:fd:91:ef:42:71:cb:11:b9:66:97:bf:ec:5b:d6:4b (ED25519)
80/tcp   open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
3306/tcp open  mysql   MySQL 5.5.5-10.11.6-MariaDB-0+deb12u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.11.6-MariaDB-0+deb12u1
|   Thread ID: 34
|   Capabilities flags: 63486
|   Some Capabilities: Speaks41ProtocolOld, SupportsLoadDataLocal, ConnectWithDatabase, Speaks41ProtocolNew, LongColumnFlag, SupportsCompression, ODBCClient, SupportsTransactions, IgnoreSigpipes, FoundRows, IgnoreSpaceBeforeParenthesis, DontAllowDatabaseTableColumn, Support41Auth, InteractiveClient, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: m!XwnCA^4)noc##&3nS4
|_  Auth Plugin Name: mysql_native_password
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.85 seconds
```
Podemos observar que el unico servicio que parece desactualizado es mysql, pero primero vamos a observar que hay el servidor apache

```bash
┌──(kali㉿kali)-[~]
└─$ whatweb 10.0.0.156
http://10.0.0.156 [200 OK] Apache[2.4.57], Country[RESERVED][ZZ], HTTPServer[Debian Linux][Apache/2.4.57 (Debian)], IP[10.0.0.156], Title[Apache2 Debian Default Page: It works]                                                                                                                                                                       
```
Y lo que hay es la pagina por defecto del servidor apache2
```html
┌──(coolatos㉿CooLaToS)-[~/HMV/jabita/ferox]
└─$ curl http://10.1.1.49/building/                                                                                                                                                                                                      1 ⨯
<!DOCTYPE html>
<html>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <link rel="stylesheet" href="https://www.w3schools.com/w3css/4/w3.css">
        <body>
          <img source
        </body>
</html>
```
