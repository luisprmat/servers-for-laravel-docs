# Servidores para Laravel en AWS
Aquí tratamos de agregar documentación que responda a la pregunta ¿Cómo desplegar un proyecto de [Laravel](https://laravel.com) en una instancia EC2 (*Elastic Computing Cloud*) de [AWS](https://aws.amazon.com) (Amazon Web Services)? y los servicios adicionales que se podrían utilizar.

Este tutorial estará en español y lo trabajaremos desde un computador con sistema operativo **Windows 11 x64 (bits)**.

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
![image](https://github.com/user-attachments/assets/3eeff7d9-26ad-461d-8b8c-f4d4d1e18bee)
1. Haga clic en el enlace EC2 en la barra superior.
Alternativamente, puede encontrarlo en el menú de *Servicios* en la categoría **Informática** (o si ya ha creado instancias EC2 en **Visitados recientemente**),
![Categoría informática](https://github.com/user-attachments/assets/7f295998-e513-4908-8acd-fc0242021dc1)
o en la barra de búsqueda:
![Barra de búsqueda](https://github.com/user-attachments/assets/2e7e0dca-4956-4b5a-b3b0-5134644fa909)
1. Ahora deberíamos estar en el panel de EC2, que se ve así:
![Panel del EC2](https://github.com/user-attachments/assets/654db17c-f84c-486a-8f9d-a6a89fe881cc)
1. En la esquina superior derecha, primero debemos elegir la región. Todas las estadísticas que se muestran en el panel corresponden a la región seleccionada, incluidos los servidores (instancias). Elegimos la que parezca más adecuada para la base de usuarios, ya que afecta la velocidad a la que se puede acceder a ella. En este ejemplo, usamos **EE.UU.Este** (Norte de Virginia) `us-east-1`.
![region de aws](https://github.com/user-attachments/assets/96599362-1cca-42e5-a8c4-abe26eed8ff7)
1. En la segunda fila del panel, hay una tarjeta llamada **Lanzar la instancia**. Los servidores se denominan instancias en EC2. Haga clic en el botón **Lanzar la instancia** para continuar.
<img src="https://github.com/user-attachments/assets/74385c6c-c136-426f-b0f7-cdbf219c0a3b" alt="lanzar la instancia" width="50%">








