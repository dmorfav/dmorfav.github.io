## Preparando un servidor debian para alojar un proyecto Symfony

Para realizar las configuraciones de el despliegue de un proyecto en PHP con el framework Symfony en un servidor Debian 10 me he montado este pequeño script que me agiliza mucho el trabajo. Les dejo los detalles debajo

Lo primero a aclarar es que el script esta pensado para que sea ejecutado con permisos de root pero no desde éste usuario, asi que no es necesario llamarlo con **sudo**. Lo primero que haremos es crear un nuevo fichero donde pegaremos el código siguiente, para ello con ejecutar en la carpeta personal ``` nano script.sh``` es suficiente.

```bash
#!/bin/bash


#*******Server packages install***********# 


# Actualizamos los repositorios de debian
sudo apt update && sudo apt upgrade -y 

# Instalamos softwares que vamos a necesitar mas adelante
sudo apt install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg -y 

# registramos la URL del repositorio de PHP8
sudo echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/sury-php.list 

 #Obtenemos la llave del repositorio para PHP
wget -qO - https://packages.sury.org/php/apt.gpg | sudo apt-key add - 

#Actualizamos la lista de paquetes
sudo apt update 

# Obtenemos el instalador de MySQL Server *** Importante siempre verificar la última versión del fichero en https://dev.mysql.com/downloads/repo/apt/ ***
wget -c https://dev.mysql.com/get/mysql-apt-config_0.8.22-1_all.deb

# instalamos el paquete de mysql config
sudo dpkg -i mysql-apt-config* 

#una vez instalado lo removemos
rm mysql-apt-config* 

#Actualizamos la lista de paquetes
sudo apt update 

# Instalamos php8.0 y un listado de extensiones que son requeridas para ejecutar un proyecto en Symfony, instalamos Apache y MySQL-Server
sudo apt install php8.0-{mysql,cli,common,xml,fpm,curl,mbstring,intl,zip} apache2 mysql-server -y 

# Activamos extensiones de Apache que son importantes
sudo a2enmod proxy_fcgi setenvif 

sudo a2enmod rewrite 

sudo a2enmod ssl 

sudo a2enconf php8.0-fpm 

# Reiniciamos el servicio de apache
sudo systemctl restart apache2 

  

#*******Technical install***********# 


# Instalamos nuestro control de versiones favorito
sudo apt install git -y 

  
# Instalamos composer para gestionar nuestras dependencias

php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" 

php -r "if (hash_file('sha384', 'composer-setup.php') === '906a84df04cea2aa72f40b5f787e49f22d4c2f19492ac310e8cba5b96ac8b64115ac402c8cd292b8a03482574915d1a8') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" 

php composer-setup.php 

php -r "unlink('composer-setup.php');" 

sudo mv composer.phar /usr/bin/composer 

# Obtenemos el cli de Symfony
wget https://get.symfony.com/cli/installer -O - | bash 

sudo mv $HOME/.symfony/bin/symfony /usr/local/bin/symfony 


#********Instalamos Node y yarn************#

#Descargamos la última versión estable de Node *** En mi caso fue la 16 pero sugiero verificarlo en https://nodejs.org/es/ *** 
curl -sL https://deb.nodesource.com/setup_16.x | sudo bash -

#Instalamos node
sudo apt install nodejs

#Instalamos yarn *** Los pasos para instalar yarn te los propone el script de instalación de node al final.

# Obtenemos la llave del repositorio
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null

#Registramos la url del repositorio
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

#Instalamos yarn
sudo apt-get update && sudo apt-get install yarn



#********Mysql server config****************# 

# Configuramos un usuario con todos los permisos para la base de datos del proyecto *** importante cambiar las credenciales y el password a utilizar será el mismo que se registró durante la instalación de MySQL-Server

mysql -u root -p"PASSWORD" -e "CREATE USER 'ADMIN'@'%' IDENTIFIED BY 'PASSWORD'"; 

mysql -u root -p"PASSWORD" -e "GRANT ALL PRIVILEGES ON *.* TO 'ADMIN'@'%'"; 

mysql -u root -p"PASSWORD" -e "FLUSH PRIVILEGES"; 

mysql -u root -p"PASSWORD" -e "grant alter,create,delete,drop,index,insert,select,update,trigger,alter routine,create routine, execute, create temporary tables on ADMIN.* to 'DATABASE'"; 

```
