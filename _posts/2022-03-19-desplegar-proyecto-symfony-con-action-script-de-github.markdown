
## Desplegar aplicación Symfony en servidor remoto utilizando los deploy de Github

>"Si la depuración es el proceso de eliminar errores, entonces la programación debe ser el proceso de introducirlos". _-- Edsger W. Dijkstra_

El despliegue automático de los proyectos es en mi opinión uno de los procesos fundamentales que deben priorizar las empresas de desarrollo de software, para llevar a cabo esto se debe tener en cuenta dos variables fundamentales, costo del despliegue y cantidad de entornos. Debemos tener presente que necesitaremos un entorno de éste tipo para cada proyecto, por lo que debemos intentar ahorrar la mayor cantidad de recursos en los entornos.

## Cantidad de entornos

En mi opinión existen tres entornos imprescindibles para el desarrollo de un proyecto:
	- Producción
	- Pre-Producción
	- Pruebas de nuevas funcionalidades 
Para el entorno de **producción** utilizaremos el código que se encuentre en la rama _**main**_ del proyecto y será el ambiente donde accederán los usuarios por lo que esta de más aclarar la importancia de no subir código con errores a esta rama. Para el entorno de pre-producción utilizaremos el código de la rama _**develop**_ y para el entorno de testing desplegaremos los pull request realizados por el equipo de desarrollo, de esta forma el Scrum Master tendrá a su disposición un herramienta más para la revisión del código que va a validar. 


## Costo del despliegue

Si utilizamos un servicio de alojamiento en la nube como [AWS](https://aws.amazon.com/) , [Google Cloud Platform](https://cloud.google.com/) , [Microsoft Azure](https://azure.microsoft.com/) u otro similar, sería bueno minimizar la cantidad de instancias que utilizaremos para lograr desplegar el entorno de desarrollo con los costes más económicos posibles. Hemos optado por la opción de **AWS** y hemos seleccionado una instancia de tipo [_**T2Medium**_](https://aws.amazon.com/es/ec2/instance-types/t2/) utilizando como referencia estos datos y seleccionando una instancia bajo demanda por hora donde el costo es de 0,0464USD genera un costo mensual aproximado de 33,4USD. 

En esta instancia desplegaremos los tres entornos apoyandonos en [Virtual Host](https://httpd.apache.org/docs/2.4/vhosts/examples.html) así no necesitaremos más de una instancia. 

## Manos a la obra

Lo primero que haremos será crear la instancia con la última versión de Debian, a la cual debemos de instalarle una serie de paquetes, para ello pueden referirse al artículo [**Preparando un servidor debian para alojar un proyecto Symfony**](https://dmorfav.github.io/2022/03/01/preparando-un-servidor-debian-para-alojar-un-proyecto-symfony.html) una vez configurado el ambiente para el despliegue debemos proceder a crear los **VirtualHost** de Apache para poder desplegar los tres entornos en la misma instancia.

### Configurando los VirtualHost de Apache

#### Virtualhost de producción

```config
<VirtualHost *:80>
ServerName PROYECTO.DOMINIO.COM
ServerAdmin admin@proyecto.com
#RewriteEngine on
#RewriteCond %{SERVER_NAME} =PROYECTO.DOMINIO.COM
#RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,QSA,R=permanent]
#</VirtualHost>
#<VirtualHost *:443>
#ServerName PROYECTO.DOMINIO.COM
#ServerAdmin admin@proyecto.com 
 DocumentRoot /var/www/html/PROYECTO/public
    <Directory /var/www/html/PROYECTO/public>
        AllowOverride All
        Order Allow,Deny
        Allow from All
        FallbackResource /index.php
    </Directory>
##Certificados SSL ##

#        SSLCertificateFile /etc/letsencrypt/live/PROYECTO.DOMINIO.COM/fullchain.pem
#        SSLCertificateKeyFile /etc/letsencrypt/live/PROYECTO.DOMINIO.COM/privkey.pem


##Logs##
       ErrorLog ${APACHE_LOG_DIR}/PROYECTO.DOMINIO.COM.com-error.log
       LogLevel warn
       CustomLog ${APACHE_LOG_DIR}/PROYECTO.DOMINIO.COM-access.log combined
</VirtualHost>
```
#### Virtualhost de pre-producción

```config
<VirtualHost *:80>
ServerName DEV_PROYECTO.DOMINIO.COM
ServerAdmin admin@proyecto.com
#RewriteEngine on
#RewriteCond %{SERVER_NAME} =DEV_PROYECTO.DOMINIO.COM
#RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,QSA,R=permanent]
#</VirtualHost>
#<VirtualHost *:443>
#ServerName DEV_PROYECTO.DOMINIO.COM
#ServerAdmin admin@proyecto.com 
 DocumentRoot /var/www/html/DEV_PROYECTO/public
    <Directory /var/www/html/DEV_PROYECTO/public>
        AllowOverride All
        Order Allow,Deny
        Allow from All
        FallbackResource /index.php
    </Directory>
##Certificados SSL ##

#        SSLCertificateFile /etc/letsencrypt/live/DEV_PROYECTO.DOMINIO.COM/fullchain.pem
#        SSLCertificateKeyFile /etc/letsencrypt/live/DEV_PROYECTO.DOMINIO.COM/privkey.pem


##Logs##
       ErrorLog ${APACHE_LOG_DIR}/DEV_PROYECTO.DOMINIO.COM.com-error.log
       LogLevel warn
       CustomLog ${APACHE_LOG_DIR}/DEV_PROYECTO.DOMINIO.COM-access.log combined
</VirtualHost>
```

#### Virtualhost de pruebas

```config
<VirtualHost *:80>
ServerName TEST_PROYECTO.DOMINIO.COM
ServerAdmin admin@proyecto.com
#RewriteEngine on
#RewriteCond %{SERVER_NAME} =TEST_PROYECTO.DOMINIO.COM
#RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,QSA,R=permanent]
#</VirtualHost>
#<VirtualHost *:443>
#ServerName TEST_PROYECTO.DOMINIO.COM
#ServerAdmin admin@proyecto.com 
 DocumentRoot /var/www/html/TEST_PROYECTO/public
    <Directory /var/www/html/TEST_PROYECTO/public>
        AllowOverride All
        Order Allow,Deny
        Allow from All
        FallbackResource /index.php
    </Directory>
##Certificados SSL ##

#        SSLCertificateFile /etc/letsencrypt/live/TEST_PROYECTO.DOMINIO.COM/fullchain.pem
#        SSLCertificateKeyFile /etc/letsencrypt/live/TEST_PROYECTO.DOMINIO.COM/privkey.pem


##Logs##
       ErrorLog ${APACHE_LOG_DIR}/TEST_PROYECTO.DOMINIO.COM.com-error.log
       LogLevel warn
       CustomLog ${APACHE_LOG_DIR}/TEST_PROYECTO.DOMINIO.COM-access.log combined
</VirtualHost>
```

Luego debemos movernos al directorio de ```/var/www/html/``` y estando ahí clonar el proyecto una vez por cada entorno, como por defecto siempre se va a clonar con el nombre del directorio **_PROYECTO_** la primera vez lo renombramos a **DEV_PROYECTO** luego a **TEST_PROYECTO** y el último lo dejamos con el nombre de **PROYECTO**. Dentro de cada directorio vamos a garantizar que esten las ramas que corresponden, para la carpeta **PROYECTO** la rama ```main``` y para los otros dos la rama ```develop```.

### Firmando con SSL 

Listo, ya tenemos los VirtualHost para desplegar los tres entornos, los cuales vamos a copiar en la carpeta ```etc/apache2/site-available``` y vamos a pasar a los sitios activos con los comandos ```a2ensite PROYECTO.conf``` ```a2ensite DEV_PROYECTO.conf``` ```a2ensite TEST_PROYECTO.conf``` y reiniciamos el servicio de **Apache** con ```systemctl restart apache2.service```. Debemos confirmar que existan los registros de DNS apuntando a la instancia, para ello puede ser utilizado el comando [```dig```](https://downloads.isc.org/isc/bind9/cur/9.17/doc/arm/html/manpages.html#dig-dns-lookup-utility) que realiza consultas DNS.

```dig @8.8.8.8 A DEV_PROYECTO.DOMINIO.COM```

```dig @8.8.8.8 A PROYECTO.DOMINIO.COM```

```dig @8.8.8.8 A TEST_PROYECTO.DOMINIO.COM```

En todos los casos debemos recibir la respuesta a la consulta con la IP de nuestra instancia

![image](https://user-images.githubusercontent.com/10134910/160295150-4cb8f120-ea25-4845-ae8f-3c1eb8771c7d.png)
>Ejemplo de consulta con dig a www.google.com

EL siguiente paso es firmar con un certificado SSL los dominios y para ello propongo utilizar el genial proyecto de [letsencrypt](https://letsencrypt.org) y lanzar el comando ``` certbot certonly``` donde de forma interactiva nos realizará varias preguntas de las cuales destacaremos esta sección:

![image](https://user-images.githubusercontent.com/10134910/160295612-8e6516f2-2d95-4eab-9909-0c4d803960a4.png)


>Importante notar que he seleccionado la opción de _webroot_ le he agregado el dominio que voy a firmar y he apuntado la dirección del proyecto a la ruta ```/var/www/html/PROYECTO/public``` contemplando que el **VirtualHost** anteriormente fue definido en la carpeta _**public**_ del proyecto. **Una vez obtenido los certificados debemos proceder a descomentar las configuraciones de los VirtualHost referentes a SSL**

### Actions de github

Ahora vamos a la última parte, para configurar los [actions](https://github.com/features/actions) en github se creará la estructura de directorios en la raíz del proyecto ```.github/workflows```. donde se agregaran los ficheros encargado de los actions.

#### Action para despliegue de la rama main
```
name: 🚀 Deploy project on main!

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: 🎉 Deploy to production
        uses: appleboy/ssh-action@master
        with:
          username: ${{ secrets.USER }}
          host: ${{ secrets.HOST }}
          key: ${{ secrets.KEY }}
          script: |
            cd /var/www/html/PROYECTO/
            git reset --hard
            rm yarn.lock package-lock.json
            git pull origin main
            sudo rm -r vendors/
            sudo rm -r var/cache/*
            composer install
            yarn install
            php bin/console doctrine:schema:update --force
            sudo chmod -R 770 var/cache/*
            sudo chmod -R 770 public/
            php bin/console cache:clear --env=prod --no-debug
            yarn encore prod     
```

#### Action para despliegue de la rama develop

```
name: 🚀 Deploy project on develop!

on:
  push:
    branches: [ develop ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: 🎉 Deploy to production
        uses: appleboy/ssh-action@master
        with:
          username: ${{ secrets.USER }}
          host: ${{ secrets.HOST }}
          key: ${{ secrets.KEY }}
          script: |
            cd /var/www/html/DEV_PROYECTO/
            git reset --hard
            rm yarn.lock package-lock.json
            git pull origin develop
            sudo rm -r vendors/
            sudo rm -r var/cache/*
            composer install
            yarn install
            php bin/console doctrine:schema:drop --force
            php bin/console doctrine:schema:create
            php bin/console doctrine:fixtures:load -n
            sudo chmod -R 770 var/cache/*
            sudo chmod -R 770 public/
            php bin/console cache:clear --env=prod --no-debug
            yarn encore prod     
```

#### Action para despliegue de la rama Testing
```
name: 🚀 Deploy project on testing!

on:
  pull_request:
    types: [review_requested]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: 🎉 Deploy to Testing
        uses: appleboy/ssh-action@master
        with:
          username: ${{ secrets.USER }}
          host: ${{ secrets.HOST }}
          key: ${{ secrets.KEY }}
          script: |
            cd /var/www/html/TEST_PROYECTO/
            git reset --hard
            rm yarn.lock package-lock.json
            git pull origin ${GITHUB_REF#refs/heads/}
            sudo rm -r vendors/
            sudo rm -r var/cache/*
            composer install
            yarn install
            php bin/console doctrine:schema:drop --force
            php bin/console doctrine:schema:create
            php bin/console doctrine:fixtures:load -n
            sudo chmod -R 770 var/cache/*
            sudo chmod -R 770 public/
            php bin/console cache:clear --env=prod --no-debug
            yarn encore prod     
```

### Conclusiones

Una vez desplegado todo el entorno entonces procedemos a probar los resultados finales, debemos tener en cuenta que cuando se abran más de un pull request todos serán desplegados en la rama de testing así que se debe tener en cuenta para las pruebas.




