# Servidores para Laravel en AWS
Aquí trato de documentar mi experiencia desplegando un proyecto de [Laravel](https://laravel.com) a una instancia EC2 (*Elastic Computing Cloud*) de [AWS](https://aws.amazon.com) (Amazon Web Services) y los servicios utilizados.

Estará en español y se trabajará desde un computador con sistema operativo **Windows 11 x64 (bits)**.

## Herramientas instaladas
Para el desarrollo y despliegue del proyecto se tienen instaladas las siguientes herramientas de software:
- [**Git**](https://git-scm.com). Obviamente para versionar el código de los proyectos y subir los cambios a este repositorio, si es la primera vez que instala git no olvidar configurar el usuario inicial:
```sh
git config --global user.name "Su Nombre Usuario"
git config --global user.email usuario@micorreo.com
```
Es importante poner el mismo correo con el que se registró en [Github](https://github.com) para que los cambios se asocien al usuario con ese correo.
- **Git Bash**. Es la terminal que usaré para acceder a las instancias EC2 por SSH (*Secure Shell*), ya que meneja la sintaxis de linux (que el SO de las instancias que usaré) y también para correr los comandos propios de laravel `php artisan <command>`, `npm run ...`, etc. Esta terminal se instala con *Git* (punto anterior). Para Windows también existen muy buenas terminales de comandos, en ese caso recomendaría instalar [Powershell ^7.4](https://learn.microsoft.com/es-es/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.4#winget) que tiene muy buenas prestaciones.
- [**Laragon**](https://laragon.org/download). (versión 6 o superior) Es una completa suite de herramientas de desarrollo para aplicaciones web que incluye:
    - *Servidores* (Apache, Nginx) Usaré [**Nginx**](https://nginx.org/en/) ya que será el mismo que instalaré en las instancias EC2.
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
