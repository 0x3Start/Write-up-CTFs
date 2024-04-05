
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

![Imagen1](https://github.com/0x3Start/Write-up-CTFs/blob/main/TheHackersLabs/Mortadela/img/VirtualBoxVM_01dQkEql5t.png?raw=true)

Ahora vamos a fuzzear en busca de algun servicio
```bash
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://10.0.0.156/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.0.0.156/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/wordpress            (Status: 301) [Size: 312] [--> http://10.0.0.156/wordpress/]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```
Vemos que nos encontramos con un wordpress, si visitamos la pagina nos encontramos con esto

![Imagen2](https://github.com/0x3Start/Write-up-CTFs/blob/main/TheHackersLabs/Mortadela/img/VirtualBoxVM_lTDXfA7RCb.png?raw=true)
Al ser un wordpress tenemos la herramienta wpscan que sirve para enumerar y buscar vulneravilidades, primero vamos a hacer un escaneo basico
```bash
┌──(kali㉿kali)-[~]
└─$ wpscan --url http://10.0.0.156/wordpress/
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.25
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.0.0.156/wordpress/ [10.0.0.156]
[+] Started: Fri Apr  5 12:45:50 2024

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.57 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.0.0.156/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.0.0.156/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://10.0.0.156/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.0.0.156/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.4.3 identified (Outdated, released on 2024-01-30).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.0.0.156/wordpress/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=6.4.3'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.0.0.156/wordpress/, Match: 'WordPress 6.4.3'

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <=======================================================================================> (137 / 137) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Fri Apr  5 12:46:23 2024
[+] Requests Done: 164
[+] Cached Requests: 4
[+] Data Sent: 43.188 KB
[+] Data Received: 280.648 KB
[+] Memory used: 227.344 MB
[+] Elapsed time: 00:00:33
```
Ha encontrado algunas cosas interesantes pero esto no nos sirve mucho, ahora vamos a enumerar los plugins que tiene el wordpress y ver su version, para eso vamos a utilizar el parametro --plugins-detection en modo agresivo, esto buscara en la carpeta de plugins de wordpress y fuzzeara en esa carpeta en busca de un codigo de estado diferente al 404
```bash
┌──(kali㉿kali)-[~]
└─$ wpscan --url http://10.0.0.156/wordpress/ --plugins-detection aggressive
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.25
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.0.0.156/wordpress/ [10.0.0.156]
[+] Started: Fri Apr  5 14:50:40 2024

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.57 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.0.0.156/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.0.0.156/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://10.0.0.156/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.0.0.156/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.4.3 identified (Outdated, released on 2024-01-30).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.0.0.156/wordpress/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=6.4.3'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.0.0.156/wordpress/, Match: 'WordPress 6.4.3'

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Aggressive Methods)
 Checking Known Locations - Time: 00:03:22 <================================================================================> (105035 / 105035) 100.00% Time: 00:03:22
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] akismet
 | Location: http://10.0.0.156/wordpress/wp-content/plugins/akismet/
 | Last Updated: 2024-03-21T00:55:00.000Z
 | Readme: http://10.0.0.156/wordpress/wp-content/plugins/akismet/readme.txt
 | [!] The version is out of date, the latest version is 5.3.2
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://10.0.0.156/wordpress/wp-content/plugins/akismet/, status: 200
 |
 | Version: 5.3.1 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://10.0.0.156/wordpress/wp-content/plugins/akismet/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://10.0.0.156/wordpress/wp-content/plugins/akismet/readme.txt

[+] wpdiscuz
 | Location: http://10.0.0.156/wordpress/wp-content/plugins/wpdiscuz/
 | Last Updated: 2024-03-27T17:26:00.000Z
 | Readme: http://10.0.0.156/wordpress/wp-content/plugins/wpdiscuz/readme.txt
 | [!] The version is out of date, the latest version is 7.6.16
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://10.0.0.156/wordpress/wp-content/plugins/wpdiscuz/, status: 200
 |
 | Version: 7.0.4 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://10.0.0.156/wordpress/wp-content/plugins/wpdiscuz/readme.txt

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:10 <=======================================================================================> (137 / 137) 100.00% Time: 00:00:10

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

```
Si analizamos los resultados podemos observar que hemos podido encontrar 2 plugins, el de "akismet" parece que no hay mucha diferencia con la version actual pero en cambio la de wpdiscuz esta bastante desactualizada
Buscando el google por alguna vulneravilidad del plugin "wpdiscuz" con la version 7.0.4 , nos encontramos bastantes exploits

![](https://github.com/0x3Start/Write-up-CTFs/blob/main/TheHackersLabs/Mortadela/img/VirtualBoxVM_XNE4QOhqRF.png?raw=true)
