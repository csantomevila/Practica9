# Práctica 9: Configuración de Apache y Bind9 con Docker Compose

En esta práctica, vamos a configurar un servidor **Apache** con soporte para dos sitios web y un servidor **DNS** utilizando **Docker Compose**. Los contenedores estarán conectados a través de una red personalizada.



## Archivos a Usar

```
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
```


## Archivo Docker-Compose

```
 Esta configuración de docker-compose.yml define un entorno con dos servicios interconectados mediante Docker: un servidor web Apache y un servidor DNS Bind9
  El objetivo principal es ofrecer un sistema funcional para alojar sitios web y resolver nombres de dominio en un entorno aislado y reproducible

El archivo docker-compose.yml define dos servicios:

Apache: Servidor web con soporte para los dos sitios.
Bind9: Servidor DNS para resolver los dominios de los sitios.
Además, define una red personalizada llamada apache_subnet.
```
```
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
```


## Configuración del servidor Apache
  ``` ContenidoVolúmenes utilizados
  El servidor Apache está diseñado para alojar dos sitios web distintos: Fábulas Maravillosas y Fábulas Oscuras. La configuración se distribuye de la siguiente manera:

    Directorio /www: Almacena los archivos HTML correspondientes a los sitios web. Cada sitio tiene su directorio raíz especificado en la configuración de Apache.
    Directorio /confApache: Contiene configuraciones personalizadas del servidor Apache, como los archivos para sitios virtuales.
```


## Contenido Archivo de configuración /var/www/fabulasmaravillosas.:
```
Sitio web 1: Fabulas Maravillosas

<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasmaravillosas.int
    ServerAlias www.fabulasmaravillosas.int
    DocumentRoot /var/www/fabulasmaravillosas
</VirtualHost>
```

## Contenido Archivo de configuración /var/www/fabulasoscuras.conf:
```
Sitio web 2: Fabulas Oscuras
Archivo de configuración fabulasoscuras.conf:


<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasoscuras.int
    ServerAlias www.fabulasoscuras.int
    DocumentRoot /var/www/fabulasoscuras
</VirtualHost>

Nota: Los archivos de configuración deben estar ubicados en el directorio confApache/sites-enabled tambien 
```

## Configuración del servidor DNS (Bind9)
```
El servidor DNS Bind9 resuelve nombres de dominio para los sitios web configurados en Apache. Utiliza los siguientes elementos:

    Volúmenes:
        confDNS/conf: Contiene configuraciones generales de Bind9.
        confDNS/zonas: Almacena los archivos de zona que definen las relaciones entre nombres de dominio y direcciones IP.
```

## Zonas de Resolucion 
```
Zona Fabulas Maravillosas
Archivo db.fabulasmaravillosas.int

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

```

```
Zona Fabulas Oscuras
Archivo de zona: db.fabulasoscuras.int

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
```

## Red personalizada
```
Ambos servicios están conectados mediante una red personalizada definida como apache_subnet. Esto permite el uso de direcciones IP estáticas en un rango específico (10.1.1.0/24) para facilitar la comunicación interna:

    Servidor Apache: 10.1.1.2
    Servidor Bind9: 10.1.1.3



networks:
  apache_subnet:
    driver: bridge
    ipam:
      config:
        - subnet: 10.1.1.0/24
```

## Instrucciones de despliegue
```
1-Clona este repositorio en tu máquina local.

2-Asegúrate de tener Docker y Docker Compose instalados.

3-Navega al directorio del proyecto.

4-Ejecuta el siguiente comando para levantar los contenedores:

docker-compose up -d

5-Accede a los sitios web en los siguientes enlaces:

http://fabulasmaravillosas
http://fabulasoscuras


"
