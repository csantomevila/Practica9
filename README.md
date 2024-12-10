# Practica9

Práctica 9: Configuración de Apache y Bind9 con Docker Compose
En esta práctica configuraremos un servidor Apache con soporte para dos sitios web y un servidor DNS utilizando Docker Compose. Los contenedores estarán conectados mediante una red personalizada.
postgres@debian:/etc/docker/Practica9$ tree /var/www/
/var/www/
├── fabulasmaravillosas
│   └── index.html
├── fabulasoscuras
│   └── index.html
└── html
    └── index.html

3 directories, 3 files
postgres@debian:/etc/docker/Practica9$ 


Contenido
Configuración del archivo docker-compose.yml
Configuración del servidor Apache
Sitio web 1: Fabulas Maravillosas
Sitio web 2: Fabulas Oscuras
Configuración del servidor DNS (Bind9)
Zona Fabulas Maravillosas
Zona Fabulas Oscuras
Red personalizada
Configuraciones adicionales
Instrucciones de despliegue
Configuración del archivo docker-compose.yml
El archivo docker-compose.yml define dos servicios: apache y bind9, además de una red personalizada llamada apache_subnet.

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
Configuración básica (fabulasmaravillosas.conf):

apache
Copiar código
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasmaravillosas.int
    ServerAlias www.fabulasmaravillosas.int
    DocumentRoot /var/www/fabulasmaravillosas
</VirtualHost>
Sitio web 2: Fabulas Oscuras
Configuración básica (fabulasoscuras.conf):

apache
Copiar código
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasoscuras.int
    ServerAlias www.fabulasoscuras.int
    DocumentRoot /var/www/fabulasoscuras
</VirtualHost>
Nota: Los archivos de configuración deben estar en el directorio confApache/sites-enabled.

Configuración del servidor DNS (Bind9)
Volúmenes utilizados
./confDNS/conf: Configuración general de Bind9.
./confDNS/zonas: Archivos de las zonas DNS.
Zona Fabulas Maravillosas
Archivo: db.fabulasmaravillosas.int

dns
Copiar código
$TTL 38400
@ IN SOA ns.fabulasmaravillosas.int. some.email.address. (
   17161018   ; serial
   10800      ; refresh
   3600       ; retry
   604800     ; expire
   38400      ; minimum
)
@ IN NS ns
ns IN A 10.1.1.3
www IN A 10.1.1.2
Zona Fabulas Oscuras
Archivo: db.fabulasoscuras.int

dns
Copiar código
$TTL 38400
@ IN SOA ns.fabulasoscuras.int. some.email.address. (
   17161017   ; serial
   10800      ; refresh
   3600       ; retry
   604800     ; expire
   38400      ; minimum
)
@ IN NS ns
ns IN A 10.1.1.3
www IN A 10.1.1.2
Red personalizada
La red apache_subnet conecta los servicios de forma aislada y con direcciones IP asignadas manualmente:

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



postgres@debian:/etc/docker/Practica9$ tree /var/www/
/var/www/
├── fabulasmaravillosas
│   └── index.html
├── fabulasoscuras
│   └── index.html
└── html
    └── index.html

3 directories, 3 files
postgres@debian:/etc/docker/Practica9$ 

  GNU nano 5.4                              /etc/hosts                                        
127.0.0.1       localhost
127.0.1.1       debian
127.0.1.1       fabulasmaravillosas
127.0.1.1       fabulasoscuras


http://fabulasoscuras
http://fabulasmaravillosas


# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters


