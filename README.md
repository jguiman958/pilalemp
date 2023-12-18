# Pila Lemp.

En este caso vamos a trabajar con nginx en vez de apache, el cual es un servidor ligero de alto rendimiento, se suele usar como proxy inverso y balanceador de carga.

# Actualizamos los repositorios.

```
apt update
```

Actualizamos repositorios.

# Instalamos nginx

```
apt install nginx -y
```
Empleamos el comando para instalar nginx.

# Instalacion de paquetes: paquete php-fpm y php-mysql

```
apt install php-fpm php-mysql -y
```

Paquetes basicos necesarios para poder conectar con una base de datos y dar funcionalidad de reindimiento optimizada.

```python
php-fpm permite mejorar el consumo de memoria con mucho tráfico, este se ejecutará como un servicio independiente de Nginx.
php-mysql permite a php interar con el sistema gestor de base de datos.
```

# Copiamos el contenido de default de default a /etc/nginx/sites-available/

```
cp ../conf/default /etc/nginx/sites-available/default
```

En el fichero default se muestra lo siguiente:

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        index index.php index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        # pass PHP scripts to FastCGI server
        #
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        location ~ /\.ht {
                deny all;
        }
}

```

Esto lo que hace es: asignar prioridad a los archivos index.php con la directiva **index** para establecer una lista de muestra de ficheros por prioridad, con el bloque **location ~ \.php$** muestra la ubicación de los ficheros de configuración *fastcgi-php.conf* y *php8.1-fpm.sock* y **location ~ /\.ht** para que los usuarios no puedan descargar los archivos **.htaccess**.

# Reiniciamos el servicio de nginx

```
systemctl restart nginx
```

Reiniciamos el servicio de nginx para que se incorporen los datos.

# Copiamos el contenido de php a html para comprobar que funciona.

```
cp ../php/info.php /var/www/html
```

Copiamos el contenido de info php al directorio principal de nginx.

# Instalación del certificado autofirmado.
# 2 Instalación de certbot y sitio web con certificado transmitido por autoridad certificadora.

<p>Pero, ¿Que es una autoridad certificadora? Nos permite que el acceso a nuestro contenido web se cifre y sea seguro, hoy dia no se permite el uso de paginas http sin el protocolo ssl/tls, con el que cifra los datos durante las peticiones.</p>

### Muestra todos los comandos que se han ejecutado.
<p>-- Aparte de que interrumpe el script en caso de errores.</p>

```
set -ex
```
## 1 Actualización de repositorios

```
 apt update
```
<p>Actualizamos los repositorios para que el software se instale correctamente y no de pié a errores durante la ejecución del script.</p>

## 2 Importamos el archivo de variables .env

```
source .env
```

<p>El cual las variables que nos interesan de este archivo son las siguientes:</p>

```
# Variables para el certificado.
CERTIFICATE_EMAIL=demo@demo.es
CERTIFICATE_DOMAIN=practica-examen1.ddns.net
```

<p>Estas son las variables que requerimos para la creación del certificado el cual a traves de una autoridad certificadora de confianza, nos va a secuerizar el contenido, mediante https, cogiendo el dominio que he creado con noip y asignando a ese dominio la ip de la máquina con la que se pretende realizar el despliegue.</p>

Esto se realizará automaticamente con certbot, ya que junto con letsencrypt se encargará el de establecer el certificado seguro.

## 3 Borramos certbot para instalarlo despues, en caso de que se encuentre, lo borramos de apt para instalarlo con snap.
#
```
apt remove certbot
```
Con esto desinstalamos certbot por si se haya en el sistema.

#Instalación de snap y actualizacion del mismo.
```
snap install core
snap refresh core
```
Instalamos y ejecutamos el gestor de paquetes snap, el cual lo necesitamos para instalar certbot.

## 4 Instalamos la aplicacion certbot

```
snap install --classic certbot
```

### Donde --classic hace que dicha aplicación se instale, con una serie de permisos para que forme parte de un entorno seguro y aislado teniendo acceso a recursos del sistema que a lo mejor no podría tener.

## 5 Creamos un alias para la aplicacion certbot

```
ln -sf /snap/bin/certbot /usr/bin/certbot
```

### Creamos un enlace simbólico donde:

```python
ln --> Para crear un enlace en el sistema.
-s --> El tipo de enlace que crea es simbólico.
-f --> Para que lo cree por la fuerza.
```
Tras eso, hemos creado un enlace simbolico en ``/usr/bin`` para que se ejecute una vez lo llamamos, es decir para que se ejecute, es necesario ya que necesitamos que se ejecute para recibir ese certificado de confianza.

## 6 Obtener el certificado.

```
certbot --apache -m $CERTIFICATE_EMAIL --agree-tos --no-eff-email -d $CERTIFICATE_DOMAIN --non-interactive
```

Nosotros si solo insertasemos ``certbot --nginx``, lo ejecutaría pero, interrumpería la automatización del script, ya que buscamos que se realice automáticamente, esto se debe a que aparecen asistentes donde hay que insertar una serie de datos.

```python
--nginx: Esto significa que da el certificado para nginx.
-m: Establecemos la direccion de correo la cual la contiene la variable $CERTIFICATE_EMAIL del archivo .env, se puede cambiar por otra.
--agree-tos: Con esto aceptamos terminos de uso.
--no-eff-email: Con esto no compartimos nuestro email con la EFF.
-d: El dominio que contiene la variable: $CERTIFICATE_DOMAIN.
--non-interactive: Para que declarar que se hace de forma no interactiva. 
```

# Instalamos wordpress mediante wp-cli (Interfaz de comandos de wordpress).
<p>Con el que podemos gestionar completamente wordpress por comandos, ya sea sus plugins, sus temas, ciertas cuestiones de seguridad, en referencia a proteger a wordpress de la red pública.</p>

## Actualización de repositorios.

<p>Primero necesitamos actualizar los repositorios para que no haya problemas con la instalación de wp-cli.</p>

```
 sudo apt update
```

# Incluimos las variables del archivo .env.

```
source .env
```
<p>Incorporamos cierta información necesaria a través de un archivo .env que dispone de credenciales necesarías para la creación automática y comoda de usuarios, bases de datos, ...</p>

### Que vamos a necesitar aquí:
Variables para la creación de ``base de datos``, para la creación del archivo ``wp-config``
```
# Configuramos variables
#-----------------------------#
WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=wp_user
WORDPRESS_DB_PASSWORD=wp_pass
IP_CLIENTE_MYSQL=localhost
WORDPRESS_DB_HOST=localhost

wordpress_title="Sitio web de IAW"
wordpress_admin_user=admin
wordpress_admin_pass=admin
wordpress_admin_email=demo@demo.es

# Variables para el certificado.
CERTIFICATE_EMAIL=demo@demo.es
CERTIFICATE_DOMAIN=practicahttps10.ddns.net
```

Tenemos que tener en cuenta el archivo default de nginx, el cual para habilitar los enlaces permanentes, hay que establecer las siguientes lineas.

```
        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                index index.php index.html index.htm;
                try_files $uri $uri/ /index.php?$args;
        }
```

Esto es lo que se debe incluir en la directiva location del fichero default, principal de nginx.

Mas adelante se explica como usar estas variables y donde se usan.

## Borramos los archivos previos.
<p>Primero vamos a borrar el archivo que descargamos del repositorio de github wp-cli.phar para poder comenzar con la instalación, el cual se pondrá en el directorio tmp.</p>
```
rm -rf /tmp/wp-cli.phar
```
El fichero wp-cli es, como explicación breve un fichero el cual agrupa muchos ficheros.
## Descargamos La utilidad wp-cli
<p>Ahora si, descargamos lo necesario para instalar wordpress, con wget lo obtenemos desde la web.</p>
```
wget https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar -P /tmp
```
<p>Y como destino "-P" lo mandamos al directorio /tmp</p>

## Asignamos permisos de ejecución al archivo wp-cli.phar
```
chmod +x /tmp/wp-cli.phar
```
Necesitamos establecer permisos de ejecución sobre el fichero, para que pueda actuar como ejecución.

## Movemos el fichero wp-cli.phar a bin para incluirlo en la lista de comandos.
Esto hará que podamos insertar el comando wp en el terminal, y poder trabajar con wordpress desde dicho terminal de comandos.
```
mv /tmp/wp-cli.phar /usr/local/bin/wp
```
Ya que bin, nos permite ejecutar comandos en linux.

## Eliminamos instalaciones previas de wordpress
```
rm -rf /var/www/html/*
```
Eliminamos de html todo el contenido al instalar wordpress.

#Descargarmos el codigo fuente de wordpress en /var/www/html

Aquí comenzamos con la descarga del codigo fuente de wordpress, como hemos dicho anteriormente wp nos permite gestionar wordpress, e incluso descargarlo para su supuesta instalación.

```
wp core download --path=/var/www/html --locale=es_ES --allow-root
```

Aquí lo que hacemos es elegir el idioma automaticamente, sin necesidad de ponerlo nosotros.

Con ``core`` nos referimos a el núcleo de WordPress contiente los archivos principales de WordPress que permiten hacer cosas como: Acceder al panel de administración de WordPress. Agregar y editar publicaciones y páginas. Administrar usuarios.

```
--path=/var/www/html --> Con esto designamos la ruta donde descargar todo el codigo fuente de wordpress.
--locale=es_ES --> Con esto configuramos el idioma de descarga.
--allow-root --> Y lo importante, tenemos que permitir que el root ejecute dicha instalación si no, presentará errores.
```

## Creamos la base de la base de datos y el usuario de la base de datos.
Aquí ya interviene el archivo .env donde emplea las siguientes variables.
```
mysql -u root <<< "DROP DATABASE IF EXISTS $WORDPRESS_DB_NAME"
mysql -u root <<< "CREATE DATABASE $WORDPRESS_DB_NAME"
mysql -u root <<< "DROP USER IF EXISTS $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL"
mysql -u root <<< "CREATE USER $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL IDENTIFIED BY '$WORDPRESS_DB_PASSWORD'"
mysql -u root <<< "GRANT ALL PRIVILEGES ON $WORDPRESS_DB_NAME.* TO $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL"
```
Estás, para ser exactos.

```
#-----------------------------#
WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=wp_user
WORDPRESS_DB_PASSWORD=wp_pass
IP_CLIENTE_MYSQL=localhost
WORDPRESS_DB_HOST=localhost
```

<p>Aquí disponemos de las variables necesarias para configurar la base de datos para wordpress, necesaría para todo despliegue, se pueden modificar las variables a antojo.</p>

## Creación del archivo wp-config 

Aquí comenzamos con la creación del archivo wp-config.

```
wp config create \
  --dbname=$WORDPRESS_DB_NAME \
  --dbuser=$WORDPRESS_DB_USER \
  --dbpass=$WORDPRESS_DB_PASSWORD \
  --path=/var/www/html \
  --allow-root
```
Aquí, se configura la base de datos el cual se menciona al usuario en cuestión, su contraseña y el nombre de la base de datos 
de wordpress.

## Instalar wordpress.

Cabe recordar que ``core`` nos permite configurar parametros de seguridad, usuarios, plugins etc. 

```
wp core install \
  --url=$CERTIFICATE_DOMAIN \
  --title="$wordpress_title" \
  --admin_user=$wordpress_admin_user \
  --admin_password=$wordpress_admin_pass \
  --admin_email=$wordpress_admin_email \
  --path=/var/www/html \
  --allow-root
```

Con esto instalamos el core de wordpress.
```
Donde contiene lo siguiente:
  --url=$CERTIFICATE_DOMAIN \ --> La variable la cual almacena nuestro dominio por el cual desplegamos nuestra aplicación web.
  --title="$wordpress_title" \ --> El titulo que le hemos puesto a nuestra pagina wordpress.
  --admin_user=$wordpress_admin_user \ --> El nombre de inicio de sesión.
  --admin_password=$wordpress_admin_pass \ --> La contraseña de inicio de sesión.
  --admin_email=$wordpress_admin_email \ --> El email...
  --path=/var/www/html \
```
Lo que estamos haciendo aquí es lo mas parecido a una instalación desatendida, olvidando el hecho de que tengamos que configurar wordpress tras la instalación, del mismo.

## Actualizamos el core
```
wp core update --path=/var/www/html --allow-root
```
Actualizamos el core, para evitar problemas a la hora de instalar plugins.., temas.., etc.
## Instalamos un tema:
```
wp theme install sydney --activate --path=/var/www/html --allow-root
```
Con este comandos, instalamos un tema el cual se necesita incorporar la ruta donde tenemos el wordpress --> /var/www/html, con --allow-root para que se inicie como root.
## Instalamos el plugin bbpress:
```
wp plugin install bbpress --activate --path=/var/www/html --allow-root
```
Aquí instalamos un plugin con la siguiente sentencia...
## Instalamos el plugin para ocultar wp-admin
Y lo mas importante es ocultar nuestro login de wordpress a los usuarios de internet para evitar, ciertas consecuencias.
```
wp plugin install wps-hide-login --activate --path=/var/www/html --allow-root
```
Se cambia por defecto a login...
## Habilitar permalinks
```
wp rewrite structure '/%postname%/' \
  --path=/var/www/html \
  --allow-root
```
Con esto realizamos un rewrite, ya que es necesario que para que las paginas ganen cierta fama para google, con objetivo de mejorar el seo, añadimos esto para que en las url aparezcan nombres en vez de parametros inconclusos.

## Modificamos automaticamente el nombre que establece por defecto el plugin wpd-hide-login
Primero que nada, podemos ver las opciones de wordpress con el siguiente comando, ya que nos interesa cambiar la opción del plugin que hemos instalado anteriormente, lo que vamos a hacer será modificar el nombre por defecto, a uno nuestro, ya que sigue siendo inseguro dejar ese por defecto.

```
wp option list --path /var/www/html --allow-root
```

Con esto vemos todas las listas de opciones incluyendo plugins, temas, etc.

```
wp option update whl_page $WORDPRESS_HIDE_LOGIN --path=/var/www/html --allow-root
```

Y con esto, he usado una variable, que incluye el dato que yo quiero insertar a esa opcion para que puede acceder al login con el nombre que le he dado.

## Copiamos el htaccess a /var/www/html
Copiamos el fichero htaccess asegurandonos de que en el archivo 000-default.conf se encuentra la directiva allowoverride All.
```
cp ../conf/.htaccess /var/www/html
```

## Cambiamos al propietario de /var/www/html como www-data
```
chown -R www-data:www-data /var/www/html
```
Cambiamos el propietario del directorio wordpress para que el usuario de apache pueda ejecutarlo.