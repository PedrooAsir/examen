# examen

# 1.

Para abrir la shell a un contenedor tenemos varias formas: 
- Con comando seria: docker exec -it [nombre del contenedor]. Ejemplo: docker exec -it servidor bash. (Bash, es el interprete en este caso)

- Por interfaz gráfica: Vamos al apartado de los contenedores, es decir, en el Visual Studio. Y en el contenedor le damos clic derecho y continuamente "Attach Shell". Hay que tener en cuenta que tienen que estar encendidos para poder hacer esto. Para ello, con un "docker start [nombre del contenedor]" y listo.

# 2.

Tenemos que arrancarlos con un docker start [nombre del contenedor] para poder hacer dichas acciones.
Ejemplo --> docker start servidor

Si no se inician, como dije anteriormente, no podras entrar a la shell de estos.

# 3.

Primero de todo, debemos crear una red nueva. Continuamente, en el archivo docker-compose hay que asignarle esa red a los contenedores.

- Para crear la red: 

docker network create \
--driver=bridge \
--subnet=55.28.0.0/16 \
--ip-range=55.28.5.0/24 \
--gateway=55.28.5.254 \
examen_subnet

* EN EL DOCKER COMPOSE PONDREMOS :

services:
  bind9:
    image: ubuntu/bind9
    container_name: asir_bind9
    ports:
      - "53:53"
    networks:
      examen_subnet:
        ipv4_address: 55.28.5.1
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
    environment:
      - TZ=Europe/Paris

  cliente:
    image: alpine
    container_name: asir_cliente
    tty: true
    stdin_open: true
    dns:
      - 55.28.5.1
    networks:
      examen_subnet:
        ipv4_address: 55.28.5.2

networks:
  examen_subnet:
    external: true


# 4.

Le añadiremos una dirección ipv4.

* Ejemplo: 

networks:
      examen_subnet:
        ipv4_address: 55.28.5.1

# 5.

Usariamos el comando: docker network inspect [nombre de la red] --> docker network inspect examen_subnet

En la salida que nos de, ya podemos ver el apartado de cada contenedor, en nuestro caso el contenedor que funciona como servidor y el otro contenedor que funciona como cliente. 

Podemos ver la informacion de la red junto a sus máquinas, básicamente.


# 6.

Su función es asignar puertos al contenedor, es decir, qué puertos tiene que usar el contenedor
del docker compose.

Ejemplo --> 8080:80 significa que el puerto 80 del contenedor se mapeará al puerto 8080.


# 7.

Redirecciona subdominios o asocia dominios con otro ya existente.

- Por ejemplo:

** Ejemplo de base de datos **:

alias   CNAME   pedro

pedro   IN A    55.28.5.1

Si hay una base de datos y tenemos el dominio "pedro" con la IP 55.28.5.4 y un
CNAME relacionandose o uniendose a ese dominio, al hacer la consulta dig al dominio (CNAME)
nos devolverá el nombre del dominio al que está apuntando, osea en este caso el nombre.

--> Lo comprobaríamos con un dig:  dig CNAME @55.28.5.1 alias.asircastelao.int
La respuesta que nos devuelve sería: "pedro.asircastelao.int"

# 8.

Necesitamos los volúmenes donde se pondrán en el fichero de docker-compose.yml. Ya que estos son archivos fuera del sistema de archivos
del contenedor que guardan su configuración aunque se elimine o se detenga.

- Ejemplo:

volumes:
- ./conf:/etc/bind
- ./zonas:/var/lib/bind

# 9.

DNS que tenga:

- www a la IP 172.16.0.1
- owncloud sea un CNAME de www
- un registro de texto con el contenido "1234ASDF"
- Comprueba que todo funciona con el comando "dig"
- Muestra en los logs que el servicio arranca correctamente

Una vez configurada la base de datos y similar:

COMPROBAMOS con un Dig, por ejemplo:

--> dig CNAME @55.28.5.1 owncloud.tiendadeelectronica.int

RESULTADO / RESPUESTA:

;; ANSWER SECTION:
owncloud.tiendadeelectronica.int. 38400 IN CNAME www.tiendadeelectronica.int.

Otro ejemplo conforme vemos que todo está correcto y funciona el dig:

--> dig -t TXT @55.28.5.1 texto.tiendadeelectronica.int

RESULTADO / RESPUESTA:

;; ANSWER SECTION:
texto.tiendadeelectronica.int. 38400 IN TXT     "1234ASDF"


Para ver los *logs*, vamos al contenedor y le damos clic derecho y "View Logs". Podemos ver que está TODO CORRECTO:

13-Nov-2023 16:59:58.446 all zones loaded
13-Nov-2023 16:59:58.446 running
13-Nov-2023 16:59:58.490 managed-keys-zone: Key 20326 for zone . is now trusted (acceptance timer complete)

# 10.

Instalamos el bind9 con "apt install bind9" y continuamente lo comprobamos con "systemctl status bind9" donde tenemos que ver que esta en verde con "running"

Configuramos todos los archivos, docker compose, base de datos...etc.

- named.conf:

include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";

- default-zones:

// prime the server with knowledge of the root servers
zone "." {
	type hint;
	file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
	type master;
	file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
	type master;
	file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
	type master;
	file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
	type master;
	file "/etc/bind/db.255";
};


- named.conf.local:

zone "ej10.int" {
	type master;
	file "/var/lib/bind/db.tiendadeelectronica.int";
	allow-query {
		any;
		};
	};

- named.conf.options
options {
	directory "/var/cache/bind";

	forwarders {
	 	8.8.8.8;
		1.1.1.1;
	 };
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};


Ponemos la máquina en adaptador puente (modo promiscuo) para poder hacer digs desde el host a una maquina y listo.


