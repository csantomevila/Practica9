# Práctica 9: Configuración de Apache y Bind9 con Docker Compose

En esta práctica, vamos a configurar un servidor **Apache** con soporte para dos sitios web y un servidor **DNS** utilizando **Docker Compose**. Los contenedores estarán conectados a través de una red personalizada.

## Estructura de Archivos

```plaintext
/var/www/
├── fabulasmaravillosas
│   └── index.html
├── fabulasoscuras
│   └── index.html
└── html
    └── index.html

## Contenido
Configuración del archivo docker-compose.yml
Configuración del servidor Apache
Sitio web 1: Fabulas Maravillosas
Sitio web 2: Fabulas Oscuras
Configuración del servidor DNS (Bind9)
Zona Fabulas Maravillosas
Zona Fabulas Oscuras
Red personalizada
Instrucciones de despliegue
Configuración del archivo docker-compose.yml
El archivo docker-compose.yml define dos servicios:

Apache: Servidor web con soporte para los dos sitios.
Bind9: Servidor DNS para resolver los dominios de los sitios.
Además, define una red personalizada llamada apache_subnet.

yaml
Copiar código
version: "3.3"

services:
  apache:
    image: php:7.4-apache
    container_name: ubuntu-apache
    ports:
      - "8080:80"
    networks:
      apache_subnet:
        ipv4_address: 10.1.1.2
    volumes:
      - ./www:/var/www/html
      - ./confApache:/etc/apache2
    command: > 
      bash -c "a2enmod mpm_prefork && apachectl start && tail -f /dev/null"
    restart: unless-stopped

  bind9:
    image: ubuntu/bind9
    container_name: asir_bind9
    ports:
      - "5300:53/tcp"
      - "5300:53/udp"
    networks:
      apache_subnet:
        ipv4_address: 10.1.1.3
    volumes:
      - ./confDNS/conf:/etc/bind
      - ./confDNS/zonas:/var/lib/bind
    restart: unless-stopped

networks:
  apache_subnet:
    driver: bridge
    ipam:
      config:
        - subnet: 10.1.1.0/24
Configuración del servidor Apache
Volúmenes utilizados
./www: Directorio donde se encuentran los archivos HTML de los sitios web.
./confApache: Configuración personalizada de Apache.
Sitio web 1: Fabulas Maravillosas
Archivo de configuración fabulasmaravillosas.conf:

apache
Copiar código
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasmaravillosas.int
    ServerAlias www.fabulasmaravillosas.int
    DocumentRoot /var/www/fabulasmaravillosas
</VirtualHost>
Sitio web 2: Fabulas Oscuras
Archivo de configuración fabulasoscuras.conf:

apache
Copiar código
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasoscuras.int
    ServerAlias www.fabulasoscuras.int
    DocumentRoot /var/www/fabulasoscuras
</VirtualHost>
Nota: Los archivos de configuración deben estar ubicados en el directorio confApache/sites-enabled.

Configuración del servidor DNS (Bind9)
Volúmenes utilizados
./confDNS/conf: Configuración general de Bind9.
./confDNS/zonas: Archivos de las zonas DNS.
Zona Fabulas Maravillosas
Archivo de zona: db.fabulasmaravillosas.int

dns
Copiar código
$TTL 38400
@   IN  SOA  ns.fabulasmaravillosas.int. some.email.address. (
                17161018 ; serial
                10800 ; refresh
                3600 ; retry
                604800 ; expire
                38400 ; minimum
)
    IN  NS   ns
    IN  A    10.1.1.3
www IN  A    10.1.1.2
Zona Fabulas Oscuras
Archivo de zona: db.fabulasoscuras.int

dns
Copiar código
$TTL 38400
@   IN  SOA  ns.fabulasoscuras.int. some.email.address. (
                17161017 ; serial
                10800 ; refresh
                3600 ; retry
                604800 ; expire
                38400 ; minimum
)
    IN  NS   ns
    IN  A    10.1.1.3
www IN  A    10.1.1.2
Red personalizada
La red apache_subnet conecta los servicios de forma aislada con direcciones IP asignadas manualmente:

Apache: 10.1.1.2
Bind9: 10.1.1.3
Definición en docker-compose.yml:

yaml
Copiar código
networks:
  apache_subnet:
    driver: bridge
    ipam:
      config:
        - subnet: 10.1.1.0/24
Instrucciones de despliegue
Clona este repositorio en tu máquina local.

Asegúrate de tener Docker y Docker Compose instalados.

Navega al directorio del proyecto.

Ejecuta el siguiente comando para levantar los contenedores:

bash
Copiar código
docker-compose up -d
Accede a los sitios web en los siguientes enlaces:

http://fabulasmaravillosas
http://fabulasoscuras
Para detener los contenedores, ejecuta:

bash
Copiar código
docker-compose down
