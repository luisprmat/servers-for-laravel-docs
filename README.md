# Servidores para Laravel en AWS
Aquí tratamos de agregar documentación que responda a la pregunta ¿Cómo desplegar un proyecto de [Laravel](https://laravel.com) en una instancia EC2 (*Elastic Computing Cloud*) de [AWS](https://aws.amazon.com) (Amazon Web Services)? y los servicios adicionales que se podrían utilizar.

Este tutorial estará en español y lo trabajaremos desde un computador con sistema operativo **Windows 11 x64 (bits)** donde subiremos una aplicación llamada `game` y consiste en un juego de triqui o tres en linea que requiere websockets para que los jugadores vean los cambios en vivo (esa parte es super interesante), por cierto este es el [repositorio](https://github.com/luisprmat/game) si se quiere ver la app.

## Herramientas instaladas
Para el desarrollo y despliegue del proyecto se tienen instaladas (localmente) las siguientes herramientas de software:
- [**Git**](https://git-scm.com). Obviamente para versionar el código de los proyectos y subir los cambios a este repositorio, si es la primera vez que instala git no olvidar configurar el usuario inicial:
```sh
git config --global user.name "Su Nombre Usuario"
git config --global user.email usuario@micorreo.com
```
Es importante poner el mismo correo con el que se registró en [Github](https://github.com) para que los cambios se asocien al usuario con ese correo.
- **Git Bash**. Es la terminal que se usará para acceder a las instancias EC2 por SSH (*Secure Shell*), ya que maneja la sintaxis de linux (que el SO de las instancias que usaré) y también para correr los comandos propios de laravel `php artisan <command>`, `npm run ...`, etc. Esta terminal se instala con *Git* (punto anterior). Para Windows también existen muy buenas terminales de comandos, en ese caso recomendaría instalar [Powershell ^7.4](https://learn.microsoft.com/es-es/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.4#winget) que tiene muy buenas prestaciones.
- [**Laragon**](https://laragon.org/download). (versión 6 o superior) Es una completa suite de herramientas de desarrollo para aplicaciones web que incluye:
    - *Servidores* (Apache, Nginx) Usaré [**Nginx**](https://nginx.org/en/) ya que será el mismo que instalaremos en las instancias EC2.
    - *Motores de base de datos* (Mysql, Postgree, Redis).
    - *Administradores de Base de datos* (HeidiSql, Adminer) - usaré [**HeidiSql**](https://www.heidisql.com) para acceder a base de datos *Sqlite* o *MySql* con host público.
    - *Terminales* (Cmder)
    - *Creación de Hosts Virtuales*. Se crean de forma automática a diferencia de XAMPP.
    Y lo más importante:
    - [*PHP*](https://www.php.net/downloads). Nuestro lenguaje estrella de programación con el que está construido *Laravel*.
    - [*Node.js*](https://nodejs.org). Para manejar todo lo relacionado con Javascript (el lenguaje del Navegador).
  
    También es bueno tener todas las herramientas en las versiones más recientes, lo cual permite hacer Laragon. A la fecha (Octubre 26 de 2024) tenemos:

    | Herramienta | Versión |
    | --- | --- |
    | PHP | 8.3.13 |
    | Nginx | 1.26.2 |
    | Mysql | 8.0.39 |

## Requisitos para los servidores de Laravel
De acuerdo a la [documentación oficial](https://laravel.com/docs/11.x/deployment#server-requirements) tenemos los siguientes requisitos para el servidor:
- PHP >= 8.2
- Ctype PHP Extension
- cURL PHP Extension
- DOM PHP Extension
- Fileinfo PHP Extension
- Filter PHP Extension
- Hash PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PCRE PHP Extension
- PDO PHP Extension
- Session PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension

Estas dependencias pueden variar dependiendo de lo que realmente necesita el proyecto o lo que la organización planea usar, pero por ahora, nos ceñiremos a la lista a continuación para ahorrar un poco de tiempo:

- NginX HTTP Server
- PHP-FPM
- Zip PHP Extension
- GD PHP Extension
- PHP CLI
- [Composer](https://getcomposer.org)
- MySQL PHP Extension
- Amazon **RDS** *para manejar* `MySQL server` (También es posible instalar `MySQL Server` directamente en la instancia **EC2**)

## Configurar una instancia EC2
1. Primero, debemos tener una cuenta de **AWS**, iniciar sesión o registrarse en https://aws.amazon.com. Omitiremos los pasos del registro ya que solo debemos seguir las instrucciones.
1. Cuando se inicie sesión debería verse algo como esto:
<br><img src="https://github.com/user-attachments/assets/3eeff7d9-26ad-461d-8b8c-f4d4d1e18bee" alt="consola aws" width="60%"><br>
1. Haga clic en el enlace EC2 en la barra superior.
Alternativamente, puede encontrarlo en el menú de *Servicios* en la categoría **Informática** (o si ya ha creado instancias EC2 en **Visitados recientemente**), o en la barra de búsqueda
<br><img src="https://github.com/user-attachments/assets/7f295998-e513-4908-8acd-fc0242021dc1" alt="Categoría informática" width="45%">
<img src="https://github.com/user-attachments/assets/2e7e0dca-4956-4b5a-b3b0-5134644fa909" alt="Barra de búsqueda" width="45%"><br>
1. Ahora deberíamos estar en el panel de EC2, que se ve así:
![Panel del EC2](https://github.com/user-attachments/assets/654db17c-f84c-486a-8f9d-a6a89fe881cc)
1. En la esquina superior derecha, primero debemos elegir la región. Todas las estadísticas que se muestran en el panel corresponden a la región seleccionada, incluidos los servidores (instancias). Elegimos la que parezca más adecuada para la base de usuarios, ya que afecta la velocidad a la que se puede acceder a ella. En este ejemplo, usamos **EE.UU.Este** (Norte de Virginia) `us-east-1`.
<br><img src="https://github.com/user-attachments/assets/96599362-1cca-42e5-a8c4-abe26eed8ff7" alt="region de aws" width="50%"><br>
1. En la segunda fila del panel, hay una tarjeta llamada **Lanzar la instancia**. Los servidores se denominan instancias en EC2. Haga clic en el botón **Lanzar la instancia** para continuar.
<br><img src="https://github.com/user-attachments/assets/74385c6c-c136-426f-b0f7-cdbf219c0a3b" alt="lanzar la instancia" width="30%"><br>
1. Ingresamos un nombre para la instancia.
<br><img src="https://github.com/user-attachments/assets/5fd08c08-f0af-46f7-808f-0b4a9298ac08" alt="Nombre de la instancia" width="70%"><br>
1. Elegimos una imagen de *Sistema Operativo*. Elegimos **Ubuntu** porque ofrece herramientas listas para usar y no necesita ninguna instalación personalizada de paquetes.
<br><img src="https://github.com/user-attachments/assets/39fcd62b-a786-48f8-8bc0-c53cd6218d1d" alt="EC2 SO" width="80%"><br>
1. Para nuestros fines, dejamos el tipo de instancia **t2.micro** sin cambios, ya que tal vez no necesitemos más potencia y además está en *el nivel gratuito*.
Según los requisitos del proyecto, es posible que cambiar esta configuración por algo más potente. Además aparecen los costos de mantenimiento de la instancia cuando se supere el nivel gratuito.
<br><img src="https://github.com/user-attachments/assets/96fe6d77-cd41-4fa1-8747-0654a78e2900" alt="Elección tipo de instancia" width="80%"><br>
1. Para acceder a la instancia creada más tarde, necesitaremos la clave SSH del servidor. Para ello debemos necesitamos **Crear un nuevo par de claves**
<br><img src="https://github.com/user-attachments/assets/17148492-5406-42d5-9a23-c13068e6ab9c" alt="Nuevo par de claves" width="80%"><br>
<img src="https://github.com/user-attachments/assets/bd5f1168-0cc7-45c3-a780-85b1f4f7936b" alt="Crear par de claves" width="60%"><br>
Cuando hagamos click en **Crear par de claves** nos solicitará guardar el archivo `ec2-ubuntu-app-server.pem` en nuestro computador, es muy importante porque esa será la identificación para poder acceder mediante *SSH* desde nuestro pc. Es buena práctica guardar este archivo en la raiz de la carpeta del usuario dentro de una carpeta "oculta" llamada `.ssh`. En nuestro caso que es *Windows* lo guardaremos en `C:\Users\%USERPROFILE%\.ssh`, en *Linux* se guarda por lo general en `/home/<user>/.ssh`.
1. En la **Configuraciones de Red**, elija **Mi IP**, para permitir el acceso solo desde su dirección IP, aunque a veces es posible que desee dejarlo desde **Cualquier lugar**, pero no se recomienda, o agregar sus reglas personalizadas. Marque **Permitir el tráfico de HTTPS desde Internet** y **Permitir el tráfico HTTP desde Internet**, ya que estamos configurando un servidor web y queremos que se acepten conexiones en los puertos 80 y 443 de forma predeterminada. Si es la primera vez que creas una instancia *EC2* posiblemente te salga algo como
<br><img src="https://github.com/user-attachments/assets/20144d77-3019-4b05-8b1e-1f8fbae59417" alt="EC2 Configuracion de red" width="80%"><br>
1. Luego deberíamos configurar el almacenamiento para agregar más volúmenes o volúmenes más grandes para los datos. No modificaremos este aspecto por ahora.
<br><img src="https://github.com/user-attachments/assets/631cebbf-890b-4f0f-aa7d-7892d0db4db7" alt="EC2 Almacenamiento" width="80%"><br>
1. No es necesario realizar cambios en la sección **Detalles avanzados**. Revisamo el resumen en la barra lateral derecha y presionamos el botón **Lanzar instancia** en la esquina inferior derecha.
<br><img src="https://github.com/user-attachments/assets/c35d93b5-d23b-4b8d-a99f-54609cd7a959" alt="EC2 Resumen" width="30%"><br>
Después de eso deberíamos ver que el lanzamiento se inició exitosamente:
<br><img src="https://github.com/user-attachments/assets/302e1eec-ab0d-4f66-b268-cc8810b6b789" alt="Iniciar instancia" width="80%"><br>

## Instalar supervisor para correr workers y websockets en producción
- Nos conectamos a la EC2
- Entramos como super usuario `sudo su -`
- Instalamos supervisor `apt install supervisor`
- Nos vamos a al directorio de configuración `cd /etc/supervisor/conf.d`
- Creamos un archivo de configuración `nano app-workers.conf`
- Escribimos la configuración

```
[program:laravel-reverb]
process_name=%(program_name)s
command=php /home/web/game/artisan reverb:start
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
numprocs=1
minfds=10000
redirect_stderr=true
stdout_logfile=/home/web/game/storage/logs/reverb.log
stopwaitsecs=3600
stdout_logfile_maxbytes=5MB
user=web
```

- Guardamos y ejecutamos `sudo supervisorctl reread` seguido de `sudo supervisorctl update`

## Agregar a un sitio un certificado SSL desde Let's Encrypt
- Nos conectamos a la EC2 por `ssh`
- Entramos como super usuario `sudo su -`
- Instalamos **CertBot** `snap install --classic certbot` obteniendo algo como

```
root@ip-172-31-91-169:~# snap install --classic certbot
certbot 2.11.0 from Certbot Project (certbot-eff✓) installed
root@ip-172-31-91-169:~#
```
- Ya que **CertBot** modifica la configuracion de `nginx` debemos guardar un respaldo del archivo de configuracion del servidor
`cp /etc/nginx/sites-available/project /etc/nginx/sites-available/project-backup`
- Corremos **CertBot**: `certbot --nginx`, obtenemos:

```
root@ip-172-31-91-169:~# certbot --nginx
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel):
```

- Agregamos  nuestra dirección de *email*
  
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.4-April-3-2024.pdf. You must agree in
order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```

- Respondemos afirmativamente **Y**

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```

- También respondemos afirmativamente **Y**

```
Account registered.

Which names would you like to activate HTTPS for?
We recommend selecting either all domains, or all domains in a VirtualHost/server block.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: triqui.luisparrado.xyz
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel):
```

- Aquí nos lista los dominios y nos pregunta a cuales desea agregar el certificado SSL, respondemos separando por comas los numerales de todos los dominios, en este caso que solo hay uno pues digitamos el numeral **1**

```
Requesting a certificate for triqui.luisparrado.xyz

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/triqui.luisparrado.xyz/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/triqui.luisparrado.xyz/privkey.pem
This certificate expires on 2025-01-27.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for triqui.luisparrado.xyz to /etc/nginx/sites-enabled/game
Congratulations! You have successfully enabled HTTPS on https://triqui.luisparrado.xyz

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
root@ip-172-31-91-169:~#
```
- Ya podemos ingresar nuevamente con el protocolo seguro *HTTPS*, ¡Genial!

## Instalar y asegurar MySQL Server 8 en Ubuntu 22.04 (EC2)
Instrucciones básicas:
- Nos conectamos a la EC2 por `ssh`
- Entramos como super usuario `sudo su -`
- Instalamos el servidor de **MySQL**: `apt install mysql-server` y esperamos que termine la instalación.

Teniendo en cuenta que el usuario `root` que viene por defecto no admite autenticación con contraseña lo que podría ser un *problema de seguridad*, ingresamos y alteramos el usuario `root` para que admita contraseña.
- Ingresamos a `mysql`
- `mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY '12345678';`, reemplazando obviamente `'12345678'` por una contraseña segura; así obtenemos algo como

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY '12345678';
Query OK, 0 rows affected (0.03 sec)

mysql>
```
- Ya podemos cerrar la conexión de **MySQL**: `mysql> exit;` y la próxima vez entrar con la contraseña (`mysql` denega el acceso si no se pone contraseña)

```
root@ip-172-31-91-169:~# mysql -u root -p
Enter password:
```
Digitamos el password asignado y

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.0.39-0ubuntu0.24.04.2 (Ubuntu)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

Siguiendo con permisos de superusuario ahora nos encargaremos de asegurar la instalación de *MySQL*, ¿porqué? primero para hacer algunas validaciones de seguridad y remover un usuario anónimo que viene por defecto con `mysql` para hacer tests, Procedamos

```
root@ip-172-31-91-169:~# mysql_secure_installation

Securing the MySQL server deployment.

Enter password for user root:
```
Digitamos la contraseña

```
VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No:
```

Aquí nos pregunta si queremos instalar el componente de validación de contraseñas el cual nos ayuda a asegurar que todas las contraseñas para todos los usuarios de *mysql* sean seguras. En esta ocasión vamos a responder con un *No* (basta con presionar **ENTER** ) para poder usar contraseñas sencillas de ejemplo pero en la realidad debiera ser un *Si*.

```
Using existing password for root.
Change the password for root ? ((Press y|Y for Yes, any other key for No) :
```

Nos pregunta si queremos cambiar la contraseña del usuario `root`, responderé *No* (basta con presionar **ENTER** ) por facilidad pero obviamente es bueno poner una contraseña segura.

```
 ... skipping.
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) :
```

Nos pregunta si queremos remover los usuarios anónimos que vienen por defecto, nuestra respuesta será afirmativa: **Y**

```
Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) :
```

Ahora nos pregunta si queremos prevenir conexiones con el usuario `root` desde fuera del servidor. Responderemos afirmativamente **Y** ya que crearemos usuarios diferentes con menos privilegios para administrar las aplicaciones y a ellos si les permitiremos conexiones externas como por ejemplo desde *HeidiSQL*

```
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : Y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) :
```

Nos pregunta si queremos quitar una base de datos llamada `test` que viene para hacer pruebas. Nuestra respuesta será afirmativa **Y**

```
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) :
```

Finalmente nos pregunta si queremos recargar los privilegios en el servidor de *MySQL*, responderemos afirmativamente (**Y**) para que nuestros cambios surtan efecto.

```
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y
Success.

All done!
root@ip-172-31-91-169:~#
```

Y ¡listo!, tenemos asegurada nuestra conexión a **MySQL**.

### Crear base de datos y usuario para nuestra aplicación
Para entrar a mysql ya no es necesario entrar como superusuario así que podemos cerrar la sesión `root` y quedamos en la sesión `ubuntu` por defecto

```
root@ip-172-31-91-169:~# exit
logout
ubuntu@ip-172-31-91-169:~$
```

Desde esta consola entramos al usuario `root` de *MySQL* (No confundir con el usuario `root` de linux): `ubuntu@ip-172-31-91-169:~$mysql -u root -p` y después de poner la contraseña estamos en el bash de `mysql>`

- Ahora procedemos a crear la base de datos para nuestra aplicación: `CREATE DATABASE game CHARACTER SET utf8 COLLATE utf8_unicode_ci;` obteniéndose

```
mysql> CREATE DATABASE game CHARACTER SET utf8 COLLATE utf8_unicode_ci;
Query OK, 1 row affected, 2 warnings (0.02 sec)
```

- Ahora procedemos a crear un usuario para nuestra aplicación: `CREATE USER 'game'@'localhost' IDENTIFIED BY '12345678';`
- Luego le asignamos a este usuario privilegios sobre todas las tablas de la base de datos creada anteriormente: `GRANT ALL PRIVILEGES ON game.* TO 'game'@'localhost';`
- Ahora corremos `FLUSH PRIVILEGES;` para limpiar la caché del servidor de *MySQL* y podemos salir de la conexión `exit;`

### Conectarse a la base de datos desde un cliente externo
Con externo nos referimos a que la conexión se hace por fuera del servidor, en este caso usaremos [HeidiSQL](https://www.heidisql.com) que viene instalado por defecto con **Laragon** en windows y usaremos SSH.
- Abrimos HeidiSQL y creamos una nueva sesión siguiendo los pasos 1, 2 y 3 de la gráfica.
![heidiSQL Nueva](https://github.com/user-attachments/assets/aa959928-b401-40e3-9c46-bf9b13572d35)
- Llenamos los datos de la pestaña **Ajustes**
    - **Tipo de Red**: MariaDB or MySQL (SSH Tunnel) -> Esta elección nos habilita la nueva pestaña **Túnel SSH**
    - **Library**: libmariadb.dll (viene por defecto)
    - **Nombre del host / IP**: localhost
    - **Usuario**: game
    - **Contraseña**: (la que se definió para este usuario en el paso anterior)
    - **Puerto**: 3306 (por defecto)
    - **Bases de datos**: game
- Ahora llenamos los datos de la pestaña **Túnel SSH**
    - **Ejecutable SSH**: ssh.exe
    - **Host SSH + Puerto**: ec2-34-201-175-185.compute-1.amazonaws.com, **Puerto**: 22
      
    Aquí copiamos la dirección *DNS de IPv4 pública* que podemos ver en las propiedades de la instancia en la consola de AWS.
    ![ec2-public-host](https://github.com/user-attachments/assets/86943256-c6d9-418e-8f49-224125729ad6)
    - **Nombre de usuario**: ubuntu
    - **Archivo de llave privada**: aqui le damos explorar y buscamos el archivo `.pem` que generamos con el par de claves de la instancia y que usamos para acceder vía SSH.
    
    Los demás campos los dejamos con sus valores por defecto y le damos click en *Abrir* guardando los cambios en la sesión.

Ahora podemos administrar la base de datos desde esta herramienta.
