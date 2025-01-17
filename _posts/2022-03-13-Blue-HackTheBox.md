---
title: Blue - HackTheBox
published: true
---

En esta ocasión os traigo la máquina Blue de HackTheBox. Es una máquina Windows. Para saber por donde se debe explotar, primero se debe realizar el escaneo. Para ello se va a star usando Nmap. Se explicará los diferentes tipos de escaneo tanto el lento cómo el rápido.

Primero se debe crear un directorio con el nombre de la máquina, después los subdirectorios para tener todo organizado. Y seguidamente lanzar una traza ICMP para saber si la máquina está activa. También se podrá mediante el Ttl ante que sistema operativo se está enfrentando.


![captura1](../Blue_Hackthebox/captura1.png)

Una vez se tiene lo necesario, se comienza con la fase de reconocimiento. Se lanza una traza ICMP. Se puede ver que la máquina está activa. En base al TTL se puede identificar que es una máquina Windows. Debido a que el TTL en máquinas Linux es 64 y en máquinas Windows es 128. En este caso disminuye una unidad debido a que se está conectado por VPN y al pasar por los nodos este disminuye una unidad. Por lo tanto siempre va a ser 127.

![captura2](../Blue_Hackthebox/captura2.png)

Con la herramienta whichSystem que está creada en el lenguaje se puede identificar a qué sistema operativo se está enfrentando. Simplemente se debe pasar la dirección IP como argumento y este nos diŕa si es Windows o linux.

![captura3](../Blue_Hackthebox/captura3.png)

Una vez identificado el sistema operativo de forma manual, se debe empezar con la enumeración de puertos. Para empezar se va a realizar el típico escaneo para detectar los puertos abierto. Si se ve que el escaneo va demasiado lento, se realizar el TCP SYN Port scan.
Primero se debe indicar que se quiere escanear todo el rango de puertos (-p-). Le indicamos que se quiere filtrar sólo por los puertos abiertos. Para añadirle velocidad mediante la plantilla de temporizado le indicamos el párametro (-T5). Le metemos el párametro (-v). Y para que no nos aplique resolución DNS le metemos el parámetro (-n).

![captura4](../Blue_Hackthebox/captur4.png)

Cómo este tipo de escaneo en máquinas Windows suele ser ciertamente lento se le va a aplicar un TCP SYN Port Scan. Para ello se le añade el parámetro (-sS) indicándole el (--min-rate 5000) para indicarle que no vaya mś lento que 5000 paquetes por segundo. Le añadimos el triple verbose para ver más información y se le añade el parámetro (-Pn) para que no aplique Host Discovery.

![captura5](../Blue_Hackthebox/captura5.png)

Lo extraemos en formato Grepeable, debido a que yo tengo una función definida en la bashrc. Esta función se llama extractPorts que lo que hace es copiarte los puertos en el teclado sin tener que poner los puertos manualmente. Lo único que hace falta es instalar 'Xclip' y 'Batcat'.

![captura7](../Blue_Hackthebox/captura7.png)

Parseamos la información del archivo extraído.

![captura8](../Blue_Hackthebox/captura8.png)

Ahora se tiene que escanear la versión y servicios que corren para estos puertos. Simplemente mediante el argumento (-sC) le indicamos que queremos lanzar una serie de scripts básicos de enumeración y con el (-sV) sirve para detectar la versión y servicios que corren para estos puertos. Para evitr poner demasiados los podemos fusionar (-sCV). Mediante el parámetro (-p) indicamos los puertos que están copiados y finalmante le pasamos la IP. Extraemos la información en un formato NMAP.

![captura9](../Blue_Hackthebox/captura9.png)

Para empezar a enumerar información podemos lanzar la herramienta 'crackmapexec' ya que el servicio SMB está expuesto. Esta herramienta lo que nos reporta es un poco más de información sobre la máquina víctima.

![captura10](../Blue_Hackthebox/captura10.png)

Cómo tiene el servicio SMB expuesto se va a proceder a escanear mediante un scripts dónde nos va a indicar si es vulnerable a EternalBlue. 

![captura11](../Blue_Hackthebox/captura11.png)

Cómo se ve que es vulnerable, vamos a proceder a descargar el exploit MS17-010 de github.

![captura12](../Blue_Hackthebox/captura12.png)

Lo clonamos mediante git y obtenemos todos los archivos necesarios.

![captura13](../Blue_Hackthebox/captura13.png)

Cómo el script es antiguo se debe ejecutar con python2, debido a que con python3 tiene unos pequeños problemas. Primero tenemos que lanzar el checker para comprobar que efectivamente es vulnerable. Pee tipo de escaneo en máquinas Windows suele ser ciertamente lento se le va a aplicar un TCP SYN Port Scan. Para ello se le añade el parámetro (-sS) indicándole el ()el archivo zzz_exploit.py y comentar unas líneas del archivo. Buscamos por 'cmd' y comentamos lo siguiente:

![captura17](../Blue_Hackthebox/captura17.png)

Ahora debemos descomentar la línea dónde se ejecuta un comando donde ponga 'cmd /c'. Y le ponemos que quiere conectarse a nuestra dirección IP ya que vamos a estar compartiendo el Netcat donde se va a entablar una conexión a nuestro equipo poniéndonos en escucha por el puerto 443 de la siguientre manera:

![captura18](../Blue_Hackthebox/captura18.png)

Una vez realizado este paso debemos poner en nuestro directorio actual donde se vaya a compartir el servicio el archivo Netcat. Simplemente lo buscamos y lo ponemos en nuestro directorio y compartimos con impacket-smbserver.

![captura19](../Blue_Hackthebox/captura19.png)

Ahora nos tenemos que poner en escucha antes de ejecutar el script.

![captura20](../Blue_Hackthebox/captura20.png)

En otra pestaña aparte, buscamos el script donde hayamos puesto el zzz_exploit y lo ejecutamos indicándole la dirección IP y un samr. Que eso se pone uno que funcina con el checker.py que se ha lanzado al principio.

![captura21](../Blue_Hackthebox/captura21.png)

Y ahora deberíamos haber ganado acceso al sistema mediante Netcat.

![captura22](../Blue_Hackthebox/captura22.png)

