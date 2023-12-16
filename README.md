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