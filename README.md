# examen

# 1.

Para abrir la shell a un contenedor tenemos varias formas: 
- Con comando seria: docker exec -it [nombre del contenedor]. Ejemplo: docker exec -it servidor bash. (Bash, es el interprete en este caso)

- Por interfaz gráfica: Vamos al apartado de los contenedores, es decir, en el Visual Studio. Y en el contenedor le damos clic derecho y continuamente "Attach Shell". Hay que tener en cuenta que tienen que estar encendidos para poder hacer esto. Para ello, con un "docker start [nombre del contenedor]" y listo.

# 2.

Tenemos que arrancarlos con un docker start [nombre del contenedor] para poder hacer dichas acciones.
Ejemplo --> docker start servidor

Cuando usas -it (docker exec **-it**) al ejecutar un contenedor, estás indicando a Docker que abra una sesión interactiva y que te permita interactuar directamente con la terminal del contenedor como si estuvieras trabajando en una máquina local.

Si no se inician, como dije anteriormente, no podras entrar a la shell o terminal de estos.

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

- En primer lugar comprobamos www.tiendadeelectronica.int:

```  

  dig @172.16.0.1 www.tiendadeelectronica.int

; <<>> DiG 9.18.18-0ubuntu0.23.04.1-Ubuntu <<>> @172.16.0.1 www.tiendadeelectronica.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10862
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 794a73974ac946e6010000006553928a9fcfbe0b17fc3e7d (good)
;; QUESTION SECTION:
;www.tiendadeelectronica.int.   IN      A

;; ANSWER SECTION:
www.tiendadeelectronica.int. 38400 IN   A       172.16.0.1

;; Query time: 0 msec
;; SERVER: 172.16.0.1#53(172.16.0.1) (UDP)
;; WHEN: Tue Nov 14 16:30:18 CET 2023
;; MSG SIZE  rcvd: 100

```

- **Dig CNAME**:
```
    dig CNAME @55.28.5.1 owncloud.tiendadeelectronica.int

  ; <<>> DiG 9.18.18-0ubuntu0.23.04.1-Ubuntu <<>> CNAME @172.16.0.1 owncloud.tiendadeelectronica.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55194
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a4e5ea54bba1b327010000006553909a2e6c9ba93e611f9f (good)
;; QUESTION SECTION:
;owncloud.tiendadeelectronica.int. IN   CNAME

;; ANSWER SECTION:
owncloud.tiendadeelectronica.int. 38400 IN CNAME www.tiendadeelectronica.int.

;; Query time: 0 msec
;; SERVER: 172.16.0.1#53(172.16.0.1) (UDP)
;; WHEN: Tue Nov 14 16:22:02 CET 2023
;; MSG SIZE  rcvd: 107
```
- __**Otro ejemplo con el registro TXT**__:
```
  - dig -t TXT @172.16.0.1 texto.tiendadeelectronica.int

  ; <<>> DiG 9.18.18-0ubuntu0.23.04.1-Ubuntu <<>> -t TXT @172.16.0.1 texto.tiendadeelectronica.int
  ; (1 server found)
  ;; global options: +cmd
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3031
  ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

  ;; OPT PSEUDOSECTION:
  ; EDNS: version: 0, flags:; udp: 1232
  ; COOKIE: d1533f449e88615b01000000655390ea8a02f2eea0dd5a43 (good)
  ;; QUESTION SECTION:
  ;texto.tiendadeelectronica.int. IN      TXT

  ;; ANSWER SECTION:
  texto.tiendadeelectronica.int. 38400 IN TXT     "1234ASDF"

  ;; Query time: 0 msec
  ;; SERVER: 172.16.0.1#53(172.16.0.1) (UDP)
  ;; WHEN: Tue Nov 14 16:23:22 CET 2023
  ;; MSG SIZE  rcvd: 107
```
- Para ver los *logs*, vamos al contenedor y le damos clic derecho y "View Logs". Podemos ver que está TODO CORRECTO:

13-Nov-2023 16:59:58.446 all zones loaded
13-Nov-2023 16:59:58.446 running
13-Nov-2023 16:59:58.490 managed-keys-zone: Key 20326 for zone . is now trusted (acceptance timer complete)

# 10.

- Instalamos el bind9 con "apt install bind9" y continuamente lo comprobamos con "systemctl status bind9" donde tenemos que ver que esta en verde con "running"

- Ahora procedemos a configurar los archivos creados en /etc/bind y /var/lib/bind (No nos olvidemos de poner mismas IP y red tanto en la máquina como en el host). 
Primero configuraremos los archivos de /etc/bind, poniendo y modificando los archivos de configuración. Después iremos a /var/lib/bind y tendremos que crear la base de datos (si todo se ha hecho bien, la carpeta de dicha ruta de bind debería estar vacía). Después de hacer esto, ponemos la máquina en adaptador puente por ejemplo en Modo Promiscuo para que otras máquinas puedan detectarla o hacer (en nuestro caso), Digs. 


- Entramos en la Terminal y tendremos que ver la IP de nuestra máquina y comprobar que está con el puerto 53 (el determinado). Lo haremos con la siguiente herramienta → “netstat -ptan” para ver nuestra interfaz DNS (IP) que se creó con “adaptador puente” con el puerto 53 (es el determinado). Finalmente, ya solo hacemos dig @[Dicha IP]:
        dig -t TXT @10.0.9.49 texto.tiendadeelectronica.int (por ejemplo) . Nos debería devolver (o solucionar) la dirección IP que haya en nuestra base de datos con ese dominio.

**Aqui tendremos la configuracion de los archivos en "/etc/bind"**:

- named.conf:
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```
- default-zones:
```
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

```
- named.conf.local:
```

zone "tiendadeelectronica.int" {
	type master;
	file "/var/lib/bind/db.tiendadeelectronica.int";
	allow-query {
		any;
		};
	};

```

```
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
```
**Creamos la base de datos en la ruta "/var/lib/bind"**

```
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.tiendadeelectronica.int. some.email.address. (
				10000002   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.tiendadeelectronica.int.
ns		IN NS 		172.16.0.2
www		IN A  		172.16.0.1
owncloud	IN CNAME	www
texto	IN TXT		"1234ASDF"

```

**UNA VEZ CONFIGURADO, REINICIAMOS PARA APLICAR LOS CAMBIOS**

Ponemos la máquina en adaptador puente (modo promiscuo) para poder hacer digs desde el host a una maquina y listo.


**DIGS**

- _Comprobación de www.tiendadeelectronica.int_:

```

dig @10.0.9.149 www.tiendadeelectronica.int

; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> @10.0.9.149 www.tiendadeelectronica.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64171
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e2e5037ecdb04d05010000006553a9c9d368c889d89b5833 (good)
;; QUESTION SECTION:
;www.tiendadeelectronica.int.	IN	A

;; ANSWER SECTION:
www.tiendadeelectronica.int. 38400 IN	A	172.16.0.1

;; Query time: 0 msec
;; SERVER: 10.0.9.149#53(10.0.9.149) (UDP)
;; WHEN: Tue Nov 14 18:09:29 CET 2023
;; MSG SIZE  rcvd: 100

```
 -  _Comprobación del registro TXT_:

```
dig -t TXT @10.0.9.149 texto.tiendadeelectronica.int

; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> -t TXT @10.0.9.149 texto.tiendadeelectronica.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40474
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 0e28bfdc2962537f010000006553aaa4941afa6833672268 (good)
;; QUESTION SECTION:
;texto.tiendadeelectronica.int.	IN	TXT

;; ANSWER SECTION:
texto.tiendadeelectronica.int. 38400 IN	TXT	"1234ASDF"

;; Query time: 4 msec
;; SERVER: 10.0.9.149#53(10.0.9.149) (UDP)
;; WHEN: Tue Nov 14 18:13:08 CET 2023
;; MSG SIZE  rcvd: 107

```

  - _Comprobación del registro CNAME_:

```

dig CNAME @10.0.9.149 owncloud.tiendadeelectronica.int

; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> CNAME @10.0.9.149 owncloud.tiendadeelectronica.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63155
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e2ccd38fb189dc64010000006553ab5091d9ef83c527aa69 (good)
;; QUESTION SECTION:
;owncloud.tiendadeelectronica.int. IN	CNAME

;; ANSWER SECTION:
owncloud.tiendadeelectronica.int. 38400	IN CNAME www.tiendadeelectronica.int.

;; Query time: 0 msec
;; SERVER: 10.0.9.149#53(10.0.9.149) (UDP)
;; WHEN: Tue Nov 14 18:16:00 CET 2023
;; MSG SIZE  rcvd: 107

```


- *COMPROBACION DE LOGS*

 - Para ver o comprobar los logs buscamos la ubicacion _/var/log/auth.log_:

 ```

--> **cat /var/log/auth.log**

Nov  7 16:10:42 pedro-VirtualBox systemd-logind[547]: New seat seat0.
Nov  7 16:10:42 pedro-VirtualBox systemd-logind[547]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:10:42 pedro-VirtualBox systemd-logind[547]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:10:42 pedro-VirtualBox systemd-logind[547]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:10:43 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:10:43 pedro-VirtualBox systemd-logind[547]: New session c1 of user gdm.


```




