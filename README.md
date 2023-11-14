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

- Dig CNAME:

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

- Otro ejemplo con el registro TXT:

  dig -t TXT @172.16.0.1 texto.tiendadeelectronica.int

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

- Para ver los *logs*, vamos al contenedor y le damos clic derecho y "View Logs". Podemos ver que está TODO CORRECTO:

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


- Creamos la base de datos en la ruta "/var/lib/bind"


- Ponemos la máquina en adaptador puente (modo promiscuo) para poder hacer digs desde el host a una maquina y listo.

;; ANSWER SECTION:
test.asircastelao.int  38400 IN  A  172.16.0.1

Para comprobar los logs buscamos la ubicacion /var/log/auth.log:

Nov  7 16:10:42 pedro-VirtualBox systemd-logind[547]: New seat seat0.
Nov  7 16:10:42 pedro-VirtualBox systemd-logind[547]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:10:42 pedro-VirtualBox systemd-logind[547]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:10:42 pedro-VirtualBox systemd-logind[547]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:10:43 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:10:43 pedro-VirtualBox systemd-logind[547]: New session c1 of user gdm.
Nov  7 16:10:43 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:10:45 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.38 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:11:09 pedro-VirtualBox dbus-daemon[500]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:11:16 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:11:16 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:11:16 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:11:16 pedro-VirtualBox systemd-logind[547]: New session 2 of user pedro.
Nov  7 16:11:16 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:11:16 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:11:17 pedro-VirtualBox gnome-keyring-daemon[1546]: The PKCS#11 component was already initialized
Nov  7 16:11:17 pedro-VirtualBox gnome-keyring-daemon[1546]: The SSH agent was already initialized
Nov  7 16:11:17 pedro-VirtualBox gnome-keyring-daemon[1546]: The Secret Service was already initialized
Nov  7 16:11:17 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.84 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:11:19 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:11:19 pedro-VirtualBox systemd-logind[547]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:11:19 pedro-VirtualBox systemd-logind[547]: Removed session c1.
Nov  7 16:11:19 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.38, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:11:29 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session closed for user gdm
Nov  7 16:11:42 pedro-VirtualBox dbus-daemon[500]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:11:51 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt update
Nov  7 16:11:51 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:12:12 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:12:20 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:12:20 pedro-VirtualBox pkexec[3164]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/usr/lib/update-notifier/package-system-locked]
Nov  7 16:13:06 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt install -y build-essential linux-headers linux-headers-6.2.0-36-generic
Nov  7 16:13:06 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:13:07 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:14:37 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt install -y build-essential linux-headers-6.2.0-36-generic
Nov  7 16:14:37 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:15:21 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:16:40 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:4022:36173 [/bin/sh /media/pedro/VBox_GAs_7.0.8/runasroot.sh --has-terminal VirtualBox Guest Additions installation /bin/sh /media/pedro/VBox\_GAs\_7\.0\.8/VBoxLinuxAdditions\.run --xwin Please try running "/media/pedro/VBox_GAs_7.0.8/VBoxLinuxAdditions.run" manually.] (owned by unix-user:pedro)
Nov  7 16:16:40 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:16:40 pedro-VirtualBox pkexec[4029]: pedro: Executing command [USER=root] [TTY=/dev/pts/2] [CWD=/media/pedro/VBox_GAs_7.0.8] [COMMAND=/bin/sh /media/pedro/VBox_GAs_7.0.8/VBoxLinuxAdditions.run --xwin]
Nov  7 16:17:01 pedro-VirtualBox CRON[12309]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
Nov  7 16:17:01 pedro-VirtualBox CRON[12309]: pam_unix(cron:session): session closed for user root
Nov  7 16:17:03 pedro-VirtualBox useradd[12325]: new user: name=vboxadd, UID=999, GID=1, home=/var/run/vboxadd, shell=/bin/false, from=/dev/pts/2
Nov  7 16:17:03 pedro-VirtualBox useradd[12332]: failed adding user 'vboxadd', data deleted
Nov  7 16:17:03 pedro-VirtualBox groupadd[12341]: group added to /etc/group: name=vboxsf, GID=999
Nov  7 16:17:03 pedro-VirtualBox groupadd[12341]: group added to /etc/gshadow: name=vboxsf
Nov  7 16:17:03 pedro-VirtualBox groupadd[12341]: new group: name=vboxsf, GID=999
Nov  7 16:17:03 pedro-VirtualBox groupadd[12348]: group added to /etc/group: name=vboxdrmipc, GID=998
Nov  7 16:17:03 pedro-VirtualBox groupadd[12348]: group added to /etc/gshadow: name=vboxdrmipc
Nov  7 16:17:03 pedro-VirtualBox groupadd[12348]: new group: name=vboxdrmipc, GID=998
Nov  7 16:17:11 pedro-VirtualBox systemd-logind[547]: System is rebooting.
Nov  7 16:17:11 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session closed for user pedro
Nov  7 16:17:18 pedro-VirtualBox systemd-logind[669]: New seat seat0.
Nov  7 16:17:18 pedro-VirtualBox systemd-logind[669]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:17:18 pedro-VirtualBox systemd-logind[669]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:17:18 pedro-VirtualBox systemd-logind[669]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:17:19 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:17:19 pedro-VirtualBox systemd-logind[669]: New session c1 of user gdm.
Nov  7 16:17:19 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:17:20 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:17:30 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:17:30 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:17:30 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:17:30 pedro-VirtualBox systemd-logind[669]: New session 2 of user pedro.
Nov  7 16:17:30 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:17:30 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:17:30 pedro-VirtualBox gnome-keyring-daemon[1478]: The SSH agent was already initialized
Nov  7 16:17:30 pedro-VirtualBox gnome-keyring-daemon[1478]: The PKCS#11 component was already initialized
Nov  7 16:17:30 pedro-VirtualBox gnome-keyring-daemon[1478]: The Secret Service was already initialized
Nov  7 16:17:31 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.78 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:17:32 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:17:32 pedro-VirtualBox systemd-logind[669]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:17:32 pedro-VirtualBox systemd-logind[669]: Removed session c1.
Nov  7 16:17:32 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:17:44 pedro-VirtualBox dbus-daemon[618]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:18:01 pedro-VirtualBox systemd-logind[669]: System is powering down.
Nov  7 16:18:34 pedro-VirtualBox systemd-logind[577]: New seat seat0.
Nov  7 16:18:34 pedro-VirtualBox systemd-logind[577]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:18:34 pedro-VirtualBox systemd-logind[577]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:18:34 pedro-VirtualBox systemd-logind[577]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:18:35 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:18:35 pedro-VirtualBox systemd-logind[577]: New session c1 of user gdm.
Nov  7 16:18:35 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:18:36 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:18:41 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:18:41 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:18:41 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:18:41 pedro-VirtualBox systemd-logind[577]: New session 2 of user pedro.
Nov  7 16:18:41 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:18:41 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:18:41 pedro-VirtualBox gnome-keyring-daemon[1406]: The SSH agent was already initialized
Nov  7 16:18:41 pedro-VirtualBox gnome-keyring-daemon[1406]: The Secret Service was already initialized
Nov  7 16:18:41 pedro-VirtualBox gnome-keyring-daemon[1406]: The PKCS#11 component was already initialized
Nov  7 16:18:42 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.79 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:18:44 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:18:44 pedro-VirtualBox systemd-logind[577]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:18:44 pedro-VirtualBox systemd-logind[577]: Removed session c1.
Nov  7 16:18:44 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:18:54 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session closed for user gdm
Nov  7 16:19:00 pedro-VirtualBox dbus-daemon[541]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:19:45 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:19:45 pedro-VirtualBox pkexec[2209]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/usr/lib/update-notifier/package-system-locked]
Nov  7 16:21:19 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt update
Nov  7 16:21:19 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:21:26 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:21:29 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt upgrade
Nov  7 16:21:29 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:23:35 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:23:46 pedro-VirtualBox systemd-logind[577]: System is rebooting.
Nov  7 16:23:54 pedro-VirtualBox systemd-logind[596]: New seat seat0.
Nov  7 16:23:54 pedro-VirtualBox systemd-logind[596]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:23:54 pedro-VirtualBox systemd-logind[596]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:23:54 pedro-VirtualBox systemd-logind[596]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:23:55 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:23:55 pedro-VirtualBox systemd-logind[596]: New session c1 of user gdm.
Nov  7 16:23:55 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:23:56 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:24:08 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:24:08 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:24:08 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:24:08 pedro-VirtualBox systemd-logind[596]: New session 2 of user pedro.
Nov  7 16:24:08 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:24:08 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:24:09 pedro-VirtualBox gnome-keyring-daemon[1405]: The SSH agent was already initialized
Nov  7 16:24:09 pedro-VirtualBox gnome-keyring-daemon[1405]: The Secret Service was already initialized
Nov  7 16:24:09 pedro-VirtualBox gnome-keyring-daemon[1405]: The PKCS#11 component was already initialized
Nov  7 16:24:09 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.78 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:24:11 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:24:11 pedro-VirtualBox systemd-logind[596]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:24:11 pedro-VirtualBox systemd-logind[596]: Removed session c1.
Nov  7 16:24:11 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:24:20 pedro-VirtualBox dbus-daemon[543]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:26:57 pedro-VirtualBox systemd-logind[596]: System is powering down.
Nov  7 16:27:57 pedro-VirtualBox systemd-logind[580]: New seat seat0.
Nov  7 16:27:57 pedro-VirtualBox systemd-logind[580]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:27:57 pedro-VirtualBox systemd-logind[580]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:27:57 pedro-VirtualBox systemd-logind[580]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:27:58 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:27:58 pedro-VirtualBox systemd-logind[580]: New session c1 of user gdm.
Nov  7 16:27:58 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:27:59 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:28:03 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:28:03 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:28:03 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:28:03 pedro-VirtualBox systemd-logind[580]: New session 2 of user pedro.
Nov  7 16:28:04 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:28:04 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:28:04 pedro-VirtualBox gnome-keyring-daemon[1417]: The SSH agent was already initialized
Nov  7 16:28:04 pedro-VirtualBox gnome-keyring-daemon[1417]: The Secret Service was already initialized
Nov  7 16:28:04 pedro-VirtualBox gnome-keyring-daemon[1417]: The PKCS#11 component was already initialized
Nov  7 16:28:05 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.77 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:28:06 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:28:06 pedro-VirtualBox systemd-logind[580]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:28:06 pedro-VirtualBox systemd-logind[580]: Removed session c1.
Nov  7 16:28:06 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:28:23 pedro-VirtualBox dbus-daemon[549]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:28:43 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt update
Nov  7 16:28:43 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:28:44 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:28:47 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt install apt-transport-https ca-certificates curl software-properties-common
Nov  7 16:28:47 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:28:52 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:28:58 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt-key add -
Nov  7 16:28:58 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:28:58 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:29:03 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/add-apt-repository 'deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable'
Nov  7 16:29:03 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:29:09 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:29:15 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt update
Nov  7 16:29:15 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:29:17 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:29:27 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt update
Nov  7 16:29:27 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:29:28 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:29:52 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt install docker-ce
Nov  7 16:29:52 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:30:01 pedro-VirtualBox CRON[4266]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
Nov  7 16:30:01 pedro-VirtualBox CRON[4266]: pam_unix(cron:session): session closed for user root
Nov  7 16:30:31 pedro-VirtualBox groupadd[4473]: group added to /etc/group: name=docker, GID=997
Nov  7 16:30:31 pedro-VirtualBox groupadd[4473]: group added to /etc/gshadow: name=docker
Nov  7 16:30:31 pedro-VirtualBox groupadd[4473]: new group: name=docker, GID=997
Nov  7 16:30:34 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:30:37 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/systemctl status docker
Nov  7 16:30:37 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:30:53 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:30:58 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/sbin/usermod -aG docker pedro
Nov  7 16:30:58 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:30:58 pedro-VirtualBox usermod[4808]: add 'pedro' to group 'docker'
Nov  7 16:30:58 pedro-VirtualBox usermod[4808]: add 'pedro' to shadow group 'docker'
Nov  7 16:30:58 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:32:02 pedro-VirtualBox systemd-logind[580]: System is rebooting.
Nov  7 16:32:09 pedro-VirtualBox systemd-logind[568]: New seat seat0.
Nov  7 16:32:09 pedro-VirtualBox systemd-logind[568]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:32:09 pedro-VirtualBox systemd-logind[568]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:32:09 pedro-VirtualBox systemd-logind[568]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:32:10 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:32:10 pedro-VirtualBox systemd-logind[568]: New session c1 of user gdm.
Nov  7 16:32:10 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:32:11 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:32:18 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:32:18 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:32:18 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:32:18 pedro-VirtualBox systemd-logind[568]: New session 2 of user pedro.
Nov  7 16:32:18 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:32:18 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:32:18 pedro-VirtualBox gnome-keyring-daemon[1609]: The SSH agent was already initialized
Nov  7 16:32:18 pedro-VirtualBox gnome-keyring-daemon[1609]: The Secret Service was already initialized
Nov  7 16:32:18 pedro-VirtualBox gnome-keyring-daemon[1609]: The PKCS#11 component was already initialized
Nov  7 16:32:19 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.82 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:32:20 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:32:20 pedro-VirtualBox systemd-logind[568]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:32:20 pedro-VirtualBox systemd-logind[568]: Removed session c1.
Nov  7 16:32:20 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:32:35 pedro-VirtualBox dbus-daemon[548]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:34:07 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/snap install code --classic
Nov  7 16:34:07 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:35:33 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:36:11 pedro-VirtualBox dbus-daemon[548]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:38:21 pedro-VirtualBox systemd-logind[568]: System is rebooting.
Nov  7 16:38:30 pedro-VirtualBox systemd-logind[649]: New seat seat0.
Nov  7 16:38:30 pedro-VirtualBox systemd-logind[649]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:38:30 pedro-VirtualBox systemd-logind[649]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:38:30 pedro-VirtualBox systemd-logind[649]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:38:31 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:38:31 pedro-VirtualBox systemd-logind[649]: New session c1 of user gdm.
Nov  7 16:38:31 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:38:32 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.41 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:38:35 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:38:35 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:38:35 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:38:35 pedro-VirtualBox systemd-logind[649]: New session 2 of user pedro.
Nov  7 16:38:35 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:38:35 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:38:36 pedro-VirtualBox gnome-keyring-daemon[1683]: The SSH agent was already initialized
Nov  7 16:38:36 pedro-VirtualBox gnome-keyring-daemon[1683]: The Secret Service was already initialized
Nov  7 16:38:36 pedro-VirtualBox gnome-keyring-daemon[1683]: The PKCS#11 component was already initialized
Nov  7 16:38:36 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.81 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:38:37 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:38:37 pedro-VirtualBox systemd-logind[649]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:38:38 pedro-VirtualBox systemd-logind[649]: Removed session c1.
Nov  7 16:38:38 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.41, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:38:56 pedro-VirtualBox dbus-daemon[628]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:41:47 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt install -y bind9
Nov  7 16:41:47 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:41:50 pedro-VirtualBox groupadd[2583]: group added to /etc/group: name=bind, GID=137
Nov  7 16:41:50 pedro-VirtualBox groupadd[2583]: group added to /etc/gshadow: name=bind
Nov  7 16:41:50 pedro-VirtualBox groupadd[2583]: new group: name=bind, GID=137
Nov  7 16:41:50 pedro-VirtualBox useradd[2591]: new user: name=bind, UID=129, GID=137, home=/var/cache/bind, shell=/usr/sbin/nologin, from=/dev/pts/2
Nov  7 16:41:50 pedro-VirtualBox usermod[2598]: change user 'bind' password
Nov  7 16:41:50 pedro-VirtualBox chage[2605]: changed password expiry for bind
Nov  7 16:41:52 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:41:57 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt update
Nov  7 16:41:57 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:41:59 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:42:01 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt install -y bind9
Nov  7 16:42:01 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:42:01 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:42:20 pedro-VirtualBox systemd-logind[649]: System is rebooting.
Nov  7 16:42:28 pedro-VirtualBox systemd-logind[646]: New seat seat0.
Nov  7 16:42:28 pedro-VirtualBox systemd-logind[646]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:42:28 pedro-VirtualBox systemd-logind[646]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:42:28 pedro-VirtualBox systemd-logind[646]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:42:29 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:42:29 pedro-VirtualBox systemd-logind[646]: New session c1 of user gdm.
Nov  7 16:42:29 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:42:30 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.41 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:42:36 pedro-VirtualBox systemd-logind[646]: System is powering down.
Nov  7 16:43:42 pedro-VirtualBox systemd-logind[658]: New seat seat0.
Nov  7 16:43:42 pedro-VirtualBox systemd-logind[658]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:43:42 pedro-VirtualBox systemd-logind[658]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:43:42 pedro-VirtualBox systemd-logind[658]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:43:43 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:43:43 pedro-VirtualBox systemd-logind[658]: New session c1 of user gdm.
Nov  7 16:43:43 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:43:44 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.41 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:43:48 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:43:48 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:43:48 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:43:48 pedro-VirtualBox systemd-logind[658]: New session 2 of user pedro.
Nov  7 16:43:48 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:43:48 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:43:49 pedro-VirtualBox gnome-keyring-daemon[1707]: The SSH agent was already initialized
Nov  7 16:43:49 pedro-VirtualBox gnome-keyring-daemon[1707]: The Secret Service was already initialized
Nov  7 16:43:49 pedro-VirtualBox gnome-keyring-daemon[1707]: The PKCS#11 component was already initialized
Nov  7 16:43:50 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.81 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:43:51 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:43:51 pedro-VirtualBox systemd-logind[658]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:43:51 pedro-VirtualBox systemd-logind[658]: Removed session c1.
Nov  7 16:43:51 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.41, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:44:08 pedro-VirtualBox dbus-daemon[640]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:44:16 pedro-VirtualBox polkit-agent-helper-1[2494]: pam_unix(polkit-1:auth): authentication failure; logname= uid=1000 euid=0 tty= ruser=pedro rhost=  user=pedro
Nov  7 16:44:20 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain TEMPORARY authorization for action org.gtk.vfs.file-operations-helper for unix-process:2490:3438 [/bin/sh -c pkexec /usr/libexec/gvfsd-admin "$@" --address $DBUS_SESSION_BUS_ADDRESS --dir $XDG_RUNTIME_DIR gvfsd-admin --spawner :1.5 /org/gtk/gvfs/exec_spaw/4] (owned by unix-user:pedro)
Nov  7 16:44:20 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:44:20 pedro-VirtualBox pkexec[2491]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/usr/libexec/gvfsd-admin --spawner :1.5 /org/gtk/gvfs/exec_spaw/4 --address unix:path=/run/user/1000/bus --dir /run/user/1000]
Nov  7 16:44:22 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain TEMPORARY authorization for action org.gtk.vfs.file-operations for unix-process:2437:2620 [/usr/bin/nautilus --gapplication-service] (owned by unix-user:pedro)
Nov  7 16:45:14 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt-get remove --auto-remove bind9
Nov  7 16:45:14 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:45:18 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:45:28 pedro-VirtualBox systemd-logind[658]: System is powering down.
Nov  7 16:48:26 pedro-VirtualBox systemd-logind[572]: New seat seat0.
Nov  7 16:48:26 pedro-VirtualBox systemd-logind[572]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:48:26 pedro-VirtualBox systemd-logind[572]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:48:26 pedro-VirtualBox systemd-logind[572]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:48:27 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:48:27 pedro-VirtualBox systemd-logind[572]: New session c1 of user gdm.
Nov  7 16:48:27 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:48:28 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:48:33 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:48:33 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:48:33 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:48:33 pedro-VirtualBox systemd-logind[572]: New session 2 of user pedro.
Nov  7 16:48:33 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:48:33 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:48:34 pedro-VirtualBox gnome-keyring-daemon[1607]: The Secret Service was already initialized
Nov  7 16:48:34 pedro-VirtualBox gnome-keyring-daemon[1607]: The SSH agent was already initialized
Nov  7 16:48:34 pedro-VirtualBox gnome-keyring-daemon[1607]: The PKCS#11 component was already initialized
Nov  7 16:48:34 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.82 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:48:36 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:48:36 pedro-VirtualBox systemd-logind[572]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:48:36 pedro-VirtualBox systemd-logind[572]: Removed session c1.
Nov  7 16:48:36 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:48:52 pedro-VirtualBox dbus-daemon[553]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:49:35 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt install -y bind9
Nov  7 16:49:35 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:49:41 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:50:20 pedro-VirtualBox systemd-logind[572]: System is rebooting.
Nov  7 16:50:27 pedro-VirtualBox systemd-logind[573]: New seat seat0.
Nov  7 16:50:27 pedro-VirtualBox systemd-logind[573]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:50:27 pedro-VirtualBox systemd-logind[573]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:50:27 pedro-VirtualBox systemd-logind[573]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:50:28 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:50:28 pedro-VirtualBox systemd-logind[573]: New session c1 of user gdm.
Nov  7 16:50:28 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:50:29 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:50:35 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:50:35 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:50:35 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:50:35 pedro-VirtualBox systemd-logind[573]: New session 2 of user pedro.
Nov  7 16:50:35 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:50:35 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:50:36 pedro-VirtualBox gnome-keyring-daemon[1620]: The SSH agent was already initialized
Nov  7 16:50:36 pedro-VirtualBox gnome-keyring-daemon[1620]: The PKCS#11 component was already initialized
Nov  7 16:50:36 pedro-VirtualBox gnome-keyring-daemon[1620]: The Secret Service was already initialized
Nov  7 16:50:36 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.81 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:50:38 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:50:38 pedro-VirtualBox systemd-logind[573]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:50:38 pedro-VirtualBox systemd-logind[573]: Removed session c1.
Nov  7 16:50:38 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:50:54 pedro-VirtualBox dbus-daemon[556]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 16:52:01 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain TEMPORARY authorization for action org.gtk.vfs.file-operations-helper for unix-process:2830:9380 [/bin/sh -c pkexec /usr/libexec/gvfsd-admin "$@" --address $DBUS_SESSION_BUS_ADDRESS --dir $XDG_RUNTIME_DIR gvfsd-admin --spawner :1.5 /org/gtk/gvfs/exec_spaw/4] (owned by unix-user:pedro)
Nov  7 16:52:01 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:52:01 pedro-VirtualBox pkexec[2831]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/usr/libexec/gvfsd-admin --spawner :1.5 /org/gtk/gvfs/exec_spaw/4 --address unix:path=/run/user/1000/bus --dir /run/user/1000]
Nov  7 16:52:03 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain TEMPORARY authorization for action org.gtk.vfs.file-operations for unix-process:2782:8622 [/usr/bin/nautilus --gapplication-service] (owned by unix-user:pedro)
Nov  7 16:52:55 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain TEMPORARY authorization for action org.gtk.vfs.file-operations for unix-process:2879:14350 [/usr/bin/nautilus --gapplication-service] (owned by unix-user:pedro)
Nov  7 16:53:36 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt-get remove --auto-remove bind9
Nov  7 16:53:36 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:53:40 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:53:42 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt-get remove --auto-remove bind9
Nov  7 16:53:42 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:53:42 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:53:48 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt install -y bind9
Nov  7 16:53:48 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 16:53:52 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 16:54:06 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain TEMPORARY authorization for action org.gtk.vfs.file-operations for unix-process:3511:20842 [/usr/bin/nautilus --gapplication-service] (owned by unix-user:pedro)
Nov  7 16:54:47 pedro-VirtualBox systemd-logind[573]: System is rebooting.
Nov  7 16:54:55 pedro-VirtualBox systemd-logind[656]: New seat seat0.
Nov  7 16:54:55 pedro-VirtualBox systemd-logind[656]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 16:54:55 pedro-VirtualBox systemd-logind[656]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 16:54:55 pedro-VirtualBox systemd-logind[656]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 16:54:56 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:54:56 pedro-VirtualBox systemd-logind[656]: New session c1 of user gdm.
Nov  7 16:54:56 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 16:54:57 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:55:07 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 16:55:07 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 16:55:07 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:55:07 pedro-VirtualBox systemd-logind[656]: New session 2 of user pedro.
Nov  7 16:55:07 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 16:55:07 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 16:55:07 pedro-VirtualBox gnome-keyring-daemon[1706]: The SSH agent was already initialized
Nov  7 16:55:07 pedro-VirtualBox gnome-keyring-daemon[1706]: The Secret Service was already initialized
Nov  7 16:55:07 pedro-VirtualBox gnome-keyring-daemon[1706]: The PKCS#11 component was already initialized
Nov  7 16:55:08 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.81 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 16:55:09 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 16:55:09 pedro-VirtualBox systemd-logind[656]: Session c1 logged out. Waiting for processes to exit.
Nov  7 16:55:09 pedro-VirtualBox systemd-logind[656]: Removed session c1.
Nov  7 16:55:09 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 16:55:22 pedro-VirtualBox dbus-daemon[639]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:10:37 pedro-VirtualBox gdm-password]: gkr-pam: unlocked login keyring
Nov  7 17:10:59 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain TEMPORARY authorization for action org.gtk.vfs.file-operations-helper for unix-process:2691:96402 [/bin/sh -c pkexec /usr/libexec/gvfsd-admin "$@" --address $DBUS_SESSION_BUS_ADDRESS --dir $XDG_RUNTIME_DIR gvfsd-admin --spawner :1.3 /org/gtk/gvfs/exec_spaw/4] (owned by unix-user:pedro)
Nov  7 17:10:59 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:10:59 pedro-VirtualBox pkexec[2692]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/usr/libexec/gvfsd-admin --spawner :1.3 /org/gtk/gvfs/exec_spaw/4 --address unix:path=/run/user/1000/bus --dir /run/user/1000]
Nov  7 17:11:01 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain TEMPORARY authorization for action org.gtk.vfs.file-operations for unix-process:2648:95865 [/usr/bin/nautilus --gapplication-service] (owned by unix-user:pedro)
Nov  7 17:12:04 pedro-VirtualBox systemd-logind[656]: System is rebooting.
Nov  7 17:12:13 pedro-VirtualBox systemd-logind[574]: New seat seat0.
Nov  7 17:12:13 pedro-VirtualBox systemd-logind[574]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 17:12:13 pedro-VirtualBox systemd-logind[574]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 17:12:13 pedro-VirtualBox systemd-logind[574]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 17:12:14 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:12:14 pedro-VirtualBox systemd-logind[574]: New session c1 of user gdm.
Nov  7 17:12:14 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:12:15 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.41 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:12:23 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 17:12:23 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 17:12:23 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:12:23 pedro-VirtualBox systemd-logind[574]: New session 2 of user pedro.
Nov  7 17:12:24 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:12:24 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 17:12:24 pedro-VirtualBox gnome-keyring-daemon[1619]: The SSH agent was already initialized
Nov  7 17:12:24 pedro-VirtualBox gnome-keyring-daemon[1619]: The Secret Service was already initialized
Nov  7 17:12:24 pedro-VirtualBox gnome-keyring-daemon[1619]: The PKCS#11 component was already initialized
Nov  7 17:12:25 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.81 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:12:26 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 17:12:26 pedro-VirtualBox systemd-logind[574]: Session c1 logged out. Waiting for processes to exit.
Nov  7 17:12:26 pedro-VirtualBox systemd-logind[574]: Removed session c1.
Nov  7 17:12:26 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.41, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 17:12:39 pedro-VirtualBox dbus-daemon[549]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:12:49 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt-get remove --auto-remove bind9
Nov  7 17:12:49 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:12:52 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 17:12:56 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/apt install -y bind9
Nov  7 17:12:56 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:13:00 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 17:15:16 pedro-VirtualBox dbus-daemon[549]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:17:01 pedro-VirtualBox CRON[3608]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
Nov  7 17:17:01 pedro-VirtualBox CRON[3608]: pam_unix(cron:session): session closed for user root
Nov  7 17:18:05 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:3902:35281 [/bin/sh -c cd "/home/pedro"; "/usr/bin/pkexec" --disable-internal-agent /bin/bash -c "echo SUDOPROMPT; \"/snap/code/145/usr/share/code/bin/code\" --file-write \"/home/pedro/.config/Code/code-elevated-wzUcQgEH\" \"/etc/bind/named.conf.options\""] (owned by unix-user:pedro)
Nov  7 17:18:05 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:18:05 pedro-VirtualBox pkexec[3903]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/bin/bash -c echo SUDOPROMPT; "/snap/code/145/usr/share/code/bin/code" --file-write "/home/pedro/.config/Code/code-elevated-wzUcQgEH" "/etc/bind/named.conf.options"]
Nov  7 17:18:15 pedro-VirtualBox dbus-daemon[549]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:18:20 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:3936:36859 [/bin/sh -c cd "/home/pedro"; "/usr/bin/pkexec" --disable-internal-agent /bin/bash -c "echo SUDOPROMPT; \"/snap/code/145/usr/share/code/bin/code\" --file-write \"/home/pedro/.config/Code/code-elevated-ZgXqR9c0\" \"/etc/bind/named.conf.local\""] (owned by unix-user:pedro)
Nov  7 17:18:20 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:18:20 pedro-VirtualBox pkexec[3937]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/bin/bash -c echo SUDOPROMPT; "/snap/code/145/usr/share/code/bin/code" --file-write "/home/pedro/.config/Code/code-elevated-ZgXqR9c0" "/etc/bind/named.conf.local"]
Nov  7 17:18:32 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:3963:37981 [/bin/sh -c cd "/home/pedro"; "/usr/bin/pkexec" --disable-internal-agent /bin/bash -c "echo SUDOPROMPT; \"/snap/code/145/usr/share/code/bin/code\" --file-write \"/home/pedro/.config/Code/code-elevated-blLzuloH\" \"/etc/bind/named.conf.default-zones\""] (owned by unix-user:pedro)
Nov  7 17:18:32 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:18:32 pedro-VirtualBox pkexec[3964]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/bin/bash -c echo SUDOPROMPT; "/snap/code/145/usr/share/code/bin/code" --file-write "/home/pedro/.config/Code/code-elevated-blLzuloH" "/etc/bind/named.conf.default-zones"]
Nov  7 17:19:59 pedro-VirtualBox systemd-logind[574]: System is rebooting.
Nov  7 17:20:06 pedro-VirtualBox systemd-logind[571]: New seat seat0.
Nov  7 17:20:06 pedro-VirtualBox systemd-logind[571]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 17:20:06 pedro-VirtualBox systemd-logind[571]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 17:20:06 pedro-VirtualBox systemd-logind[571]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 17:20:07 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:20:07 pedro-VirtualBox systemd-logind[571]: New session c1 of user gdm.
Nov  7 17:20:07 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:20:08 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:20:13 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 17:20:13 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 17:20:13 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:20:13 pedro-VirtualBox systemd-logind[571]: New session 2 of user pedro.
Nov  7 17:20:13 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:20:13 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 17:20:14 pedro-VirtualBox gnome-keyring-daemon[1623]: The PKCS#11 component was already initialized
Nov  7 17:20:14 pedro-VirtualBox gnome-keyring-daemon[1623]: The Secret Service was already initialized
Nov  7 17:20:14 pedro-VirtualBox gnome-keyring-daemon[1623]: The SSH agent was already initialized
Nov  7 17:20:14 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.82 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:20:16 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 17:20:16 pedro-VirtualBox systemd-logind[571]: Session c1 logged out. Waiting for processes to exit.
Nov  7 17:20:16 pedro-VirtualBox systemd-logind[571]: Removed session c1.
Nov  7 17:20:16 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 17:20:33 pedro-VirtualBox dbus-daemon[551]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:21:10 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/var/lib ; USER=root ; COMMAND=/usr/bin/systemctl status bind9
Nov  7 17:21:10 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:21:16 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 17:22:00 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/var/lib ; USER=root ; COMMAND=/usr/bin/apt purge bind9
Nov  7 17:22:00 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:22:04 pedro-VirtualBox userdel[2805]: delete user 'bind'
Nov  7 17:22:04 pedro-VirtualBox userdel[2805]: removed group 'bind' owned by 'bind'
Nov  7 17:22:04 pedro-VirtualBox userdel[2805]: removed shadow group 'bind' owned by 'bind'
Nov  7 17:22:05 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 17:22:13 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/var/lib ; USER=root ; COMMAND=/usr/bin/apt install bind9
Nov  7 17:22:13 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:22:16 pedro-VirtualBox groupadd[2955]: group added to /etc/group: name=bind, GID=137
Nov  7 17:22:16 pedro-VirtualBox groupadd[2955]: group added to /etc/gshadow: name=bind
Nov  7 17:22:16 pedro-VirtualBox groupadd[2955]: new group: name=bind, GID=137
Nov  7 17:22:16 pedro-VirtualBox useradd[2963]: new user: name=bind, UID=129, GID=137, home=/var/cache/bind, shell=/usr/sbin/nologin, from=/dev/pts/2
Nov  7 17:22:16 pedro-VirtualBox usermod[2970]: change user 'bind' password
Nov  7 17:22:16 pedro-VirtualBox chage[2977]: changed password expiry for bind
Nov  7 17:22:18 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 17:22:34 pedro-VirtualBox systemd-logind[571]: System is rebooting.
Nov  7 17:22:41 pedro-VirtualBox systemd-logind[575]: New seat seat0.
Nov  7 17:22:41 pedro-VirtualBox systemd-logind[575]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 17:22:41 pedro-VirtualBox systemd-logind[575]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 17:22:41 pedro-VirtualBox systemd-logind[575]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 17:22:42 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:22:42 pedro-VirtualBox systemd-logind[575]: New session c1 of user gdm.
Nov  7 17:22:42 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:22:43 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.37 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:22:56 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 17:22:56 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 17:22:56 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:22:56 pedro-VirtualBox systemd-logind[575]: New session 2 of user pedro.
Nov  7 17:22:56 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:22:56 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 17:22:57 pedro-VirtualBox gnome-keyring-daemon[1626]: The Secret Service was already initialized
Nov  7 17:22:57 pedro-VirtualBox gnome-keyring-daemon[1626]: The SSH agent was already initialized
Nov  7 17:22:57 pedro-VirtualBox gnome-keyring-daemon[1626]: The PKCS#11 component was already initialized
Nov  7 17:22:58 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.84 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:22:59 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 17:22:59 pedro-VirtualBox systemd-logind[575]: Session c1 logged out. Waiting for processes to exit.
Nov  7 17:22:59 pedro-VirtualBox systemd-logind[575]: Removed session c1.
Nov  7 17:22:59 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.37, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 17:23:08 pedro-VirtualBox dbus-daemon[557]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:28:21 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/var/lib/bind ; USER=root ; COMMAND=/usr/bin/touch db.asircastelao.int
Nov  7 17:28:21 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:28:21 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 17:29:42 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/var/lib/bind ; USER=root ; COMMAND=/usr/bin/chmod 777 db.asircastelao.int
Nov  7 17:29:42 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:29:42 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 17:30:01 pedro-VirtualBox CRON[2563]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
Nov  7 17:30:01 pedro-VirtualBox CRON[2563]: pam_unix(cron:session): session closed for user root
Nov  7 17:30:40 pedro-VirtualBox dbus-daemon[557]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:30:46 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:2801:18570 [/bin/sh -c cd "/home/pedro"; "/usr/bin/pkexec" --disable-internal-agent /bin/bash -c "echo SUDOPROMPT; \"/snap/code/145/usr/share/code/bin/code\" --file-write \"/home/pedro/.config/Code/code-elevated-wyiWHr5p\" \"/etc/bind/named.conf.default-zones\""] (owned by unix-user:pedro)
Nov  7 17:30:47 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:30:47 pedro-VirtualBox pkexec[2802]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/bin/bash -c echo SUDOPROMPT; "/snap/code/145/usr/share/code/bin/code" --file-write "/home/pedro/.config/Code/code-elevated-wyiWHr5p" "/etc/bind/named.conf.default-zones"]
Nov  7 17:30:57 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:2837:19658 [/bin/sh -c cd "/home/pedro"; "/usr/bin/pkexec" --disable-internal-agent /bin/bash -c "echo SUDOPROMPT; \"/snap/code/145/usr/share/code/bin/code\" --file-write \"/home/pedro/.config/Code/code-elevated-2aEN7qin\" \"/etc/bind/named.conf.local\""] (owned by unix-user:pedro)
Nov  7 17:30:58 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:30:58 pedro-VirtualBox pkexec[2838]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/bin/bash -c echo SUDOPROMPT; "/snap/code/145/usr/share/code/bin/code" --file-write "/home/pedro/.config/Code/code-elevated-2aEN7qin" "/etc/bind/named.conf.local"]
Nov  7 17:31:09 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:2869:20872 [/bin/sh -c cd "/home/pedro"; "/usr/bin/pkexec" --disable-internal-agent /bin/bash -c "echo SUDOPROMPT; \"/snap/code/145/usr/share/code/bin/code\" --file-write \"/home/pedro/.config/Code/code-elevated-LA8A0a6X\" \"/etc/bind/named.conf.options\""] (owned by unix-user:pedro)
Nov  7 17:31:10 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:31:10 pedro-VirtualBox pkexec[2870]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/bin/bash -c echo SUDOPROMPT; "/snap/code/145/usr/share/code/bin/code" --file-write "/home/pedro/.config/Code/code-elevated-LA8A0a6X" "/etc/bind/named.conf.options"]
Nov  7 17:33:10 pedro-VirtualBox systemd-logind[575]: System is rebooting.
Nov  7 17:33:19 pedro-VirtualBox systemd-logind[577]: New seat seat0.
Nov  7 17:33:19 pedro-VirtualBox systemd-logind[577]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 17:33:19 pedro-VirtualBox systemd-logind[577]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 17:33:19 pedro-VirtualBox systemd-logind[577]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 17:33:20 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:33:20 pedro-VirtualBox systemd-logind[577]: New session c1 of user gdm.
Nov  7 17:33:20 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:33:21 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.41 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:33:44 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 17:33:44 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 17:33:44 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:33:44 pedro-VirtualBox systemd-logind[577]: New session 2 of user pedro.
Nov  7 17:33:45 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:33:45 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 17:33:45 pedro-VirtualBox gnome-keyring-daemon[1631]: The Secret Service was already initialized
Nov  7 17:33:45 pedro-VirtualBox gnome-keyring-daemon[1631]: The SSH agent was already initialized
Nov  7 17:33:45 pedro-VirtualBox gnome-keyring-daemon[1631]: The PKCS#11 component was already initialized
Nov  7 17:33:45 pedro-VirtualBox dbus-daemon[557]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:33:46 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.80 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:33:47 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 17:33:47 pedro-VirtualBox systemd-logind[577]: Session c1 logged out. Waiting for processes to exit.
Nov  7 17:33:47 pedro-VirtualBox systemd-logind[577]: Removed session c1.
Nov  7 17:33:47 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.41, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 17:34:10 pedro-VirtualBox dbus-daemon[557]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:34:51 pedro-VirtualBox sudo:    pedro : TTY=pts/1 ; PWD=/etc/bind ; USER=root ; COMMAND=/usr/bin/touch docker-compose.yml
Nov  7 17:34:51 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:34:51 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 17:35:18 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:2715:12015 [/bin/sh -c cd "/home/pedro"; "/usr/bin/pkexec" --disable-internal-agent /bin/bash -c "echo SUDOPROMPT; \"/snap/code/145/usr/share/code/bin/code\" --file-write \"/home/pedro/.config/Code/code-elevated-wpT8IXzy\" \"/etc/bind/docker-compose.yml\""] (owned by unix-user:pedro)
Nov  7 17:35:18 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:35:18 pedro-VirtualBox pkexec[2716]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/bin/bash -c echo SUDOPROMPT; "/snap/code/145/usr/share/code/bin/code" --file-write "/home/pedro/.config/Code/code-elevated-wpT8IXzy" "/etc/bind/docker-compose.yml"]
Nov  7 17:35:26 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:2743:12847 [/bin/sh -c cd "/home/pedro"; "/usr/bin/pkexec" --disable-internal-agent /bin/bash -c "echo SUDOPROMPT; \"/snap/code/145/usr/share/code/bin/code\" --file-write \"/home/pedro/.config/Code/code-elevated-pZccl6pu\" \"/etc/bind/docker-compose.yml\""] (owned by unix-user:pedro)
Nov  7 17:35:26 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:35:26 pedro-VirtualBox pkexec[2744]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/bin/bash -c echo SUDOPROMPT; "/snap/code/145/usr/share/code/bin/code" --file-write "/home/pedro/.config/Code/code-elevated-pZccl6pu" "/etc/bind/docker-compose.yml"]
Nov  7 17:35:44 pedro-VirtualBox sudo:    pedro : TTY=pts/1 ; PWD=/etc/bind ; USER=root ; COMMAND=/usr/bin/chmod 777 docker-compose.yml
Nov  7 17:35:44 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 17:35:44 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 17:36:26 pedro-VirtualBox dbus-daemon[557]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:37:03 pedro-VirtualBox systemd-logind[577]: System is rebooting.
Nov  7 17:37:11 pedro-VirtualBox systemd-logind[643]: New seat seat0.
Nov  7 17:37:11 pedro-VirtualBox systemd-logind[643]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 17:37:11 pedro-VirtualBox systemd-logind[643]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 17:37:11 pedro-VirtualBox systemd-logind[643]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 17:37:12 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:37:12 pedro-VirtualBox systemd-logind[643]: New session c1 of user gdm.
Nov  7 17:37:12 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:37:13 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.41 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:37:25 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 17:37:25 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 17:37:25 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:37:25 pedro-VirtualBox systemd-logind[643]: New session 2 of user pedro.
Nov  7 17:37:25 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:37:25 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 17:37:25 pedro-VirtualBox gnome-keyring-daemon[1694]: The SSH agent was already initialized
Nov  7 17:37:25 pedro-VirtualBox gnome-keyring-daemon[1694]: The Secret Service was already initialized
Nov  7 17:37:25 pedro-VirtualBox gnome-keyring-daemon[1694]: The PKCS#11 component was already initialized
Nov  7 17:37:26 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.81 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:37:28 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 17:37:28 pedro-VirtualBox systemd-logind[643]: Session c1 logged out. Waiting for processes to exit.
Nov  7 17:37:28 pedro-VirtualBox systemd-logind[643]: Removed session c1.
Nov  7 17:37:28 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.41, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 17:37:37 pedro-VirtualBox dbus-daemon[625]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:38:04 pedro-VirtualBox dbus-daemon[625]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 17:53:47 pedro-VirtualBox systemd-logind[643]: System is powering down.
Nov  7 17:56:27 pedro-VirtualBox systemd-logind[569]: New seat seat0.
Nov  7 17:56:27 pedro-VirtualBox systemd-logind[569]: Watching system buttons on /dev/input/event0 (Power Button)
Nov  7 17:56:27 pedro-VirtualBox systemd-logind[569]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov  7 17:56:27 pedro-VirtualBox systemd-logind[569]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov  7 17:56:28 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:56:28 pedro-VirtualBox systemd-logind[569]: New session c1 of user gdm.
Nov  7 17:56:28 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov  7 17:56:29 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:56:34 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov  7 17:56:34 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov  7 17:56:34 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:56:34 pedro-VirtualBox systemd-logind[569]: New session 2 of user pedro.
Nov  7 17:56:34 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov  7 17:56:34 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov  7 17:56:35 pedro-VirtualBox gnome-keyring-daemon[1658]: The SSH agent was already initialized
Nov  7 17:56:35 pedro-VirtualBox gnome-keyring-daemon[1658]: The Secret Service was already initialized
Nov  7 17:56:35 pedro-VirtualBox gnome-keyring-daemon[1658]: The PKCS#11 component was already initialized
Nov  7 17:56:36 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.79 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov  7 17:56:37 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov  7 17:56:37 pedro-VirtualBox systemd-logind[569]: Session c1 logged out. Waiting for processes to exit.
Nov  7 17:56:37 pedro-VirtualBox systemd-logind[569]: Removed session c1.
Nov  7 17:56:37 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov  7 17:56:53 pedro-VirtualBox dbus-daemon[550]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 18:05:32 pedro-VirtualBox dbus-daemon[550]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 18:12:19 pedro-VirtualBox sudo:    pedro : TTY=pts/3 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/su
Nov  7 18:12:19 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 18:12:19 pedro-VirtualBox su: (to root) root on pts/0
Nov  7 18:12:19 pedro-VirtualBox su: pam_unix(su:session): session opened for user root(uid=0) by pedro(uid=0)
Nov  7 18:12:29 pedro-VirtualBox su: pam_unix(su:session): session closed for user root
Nov  7 18:12:29 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 18:12:35 pedro-VirtualBox su: (to pedro) pedro on pts/3
Nov  7 18:12:35 pedro-VirtualBox su: pam_unix(su:session): session opened for user pedro(uid=1000) by (uid=1000)
Nov  7 18:12:51 pedro-VirtualBox su: pam_unix(su:auth): authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/3 ruser=pedro rhost=  user=root
Nov  7 18:12:53 pedro-VirtualBox su: FAILED SU (to root) pedro on pts/3
Nov  7 18:12:59 pedro-VirtualBox su: pam_unix(su-l:auth): authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/3 ruser=pedro rhost=  user=root
Nov  7 18:13:01 pedro-VirtualBox su: FAILED SU (to root) pedro on pts/3
Nov  7 18:13:10 pedro-VirtualBox su: pam_unix(su-l:auth): authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/3 ruser=pedro rhost=  user=root
Nov  7 18:13:12 pedro-VirtualBox su: FAILED SU (to root) pedro on pts/3
Nov  7 18:13:16 pedro-VirtualBox su: pam_unix(su-l:auth): authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/3 ruser=pedro rhost=  user=root
Nov  7 18:13:18 pedro-VirtualBox su: FAILED SU (to root) pedro on pts/3
Nov  7 18:13:20 pedro-VirtualBox su: pam_unix(su-l:auth): authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/3 ruser=pedro rhost=  user=root
Nov  7 18:13:22 pedro-VirtualBox su: FAILED SU (to root) pedro on pts/3
Nov  7 18:13:31 pedro-VirtualBox sudo:    pedro : TTY=pts/3 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/passwd root
Nov  7 18:13:31 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 18:14:02 pedro-VirtualBox passwd[6150]: pam_pwquality(passwd:chauthtok): conversation failed
Nov  7 18:14:02 pedro-VirtualBox passwd[6150]: pam_pwquality(passwd:chauthtok): conversation failed
Nov  7 18:14:02 pedro-VirtualBox passwd[6150]: pam_pwquality(passwd:chauthtok): user aborted password change
Nov  7 18:14:02 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 18:14:02 pedro-VirtualBox su: pam_unix(su:session): session closed for user pedro
Nov  7 18:14:10 pedro-VirtualBox sudo: pam_unix(sudo:auth): authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/0 ruser=pedro rhost=  user=pedro
Nov  7 18:14:20 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/bin/passwd root
Nov  7 18:14:20 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 18:14:27 pedro-VirtualBox passwd[6195]: pam_unix(passwd:chauthtok): password changed for root
Nov  7 18:14:27 pedro-VirtualBox passwd[6195]: gkr-pam: couldn't update the login keyring password: no old password was entered
Nov  7 18:14:27 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov  7 18:14:34 pedro-VirtualBox su: pam_unix(su-l:auth): authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/0 ruser=pedro rhost=  user=root
Nov  7 18:14:36 pedro-VirtualBox su: FAILED SU (to root) pedro on pts/0
Nov  7 18:14:52 pedro-VirtualBox su: (to root) pedro on pts/0
Nov  7 18:14:52 pedro-VirtualBox su: pam_unix(su-l:session): session opened for user root(uid=0) by (uid=1000)
Nov  7 18:17:01 pedro-VirtualBox CRON[6728]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
Nov  7 18:17:01 pedro-VirtualBox CRON[6728]: pam_unix(cron:session): session closed for user root
Nov  7 18:30:01 pedro-VirtualBox CRON[6899]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
Nov  7 18:30:01 pedro-VirtualBox CRON[6899]: pam_unix(cron:session): session closed for user root
Nov  7 18:38:26 pedro-VirtualBox gdm-password]: gkr-pam: unlocked login keyring
Nov  7 18:39:35 pedro-VirtualBox su: pam_unix(su-l:session): session closed for user root
Nov  7 18:40:02 pedro-VirtualBox dbus-daemon[550]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov  7 18:46:39 pedro-VirtualBox systemd-logind[569]: System is powering down.
Nov 13 15:25:26 pedro-VirtualBox systemd-logind[573]: New seat seat0.
Nov 13 15:25:26 pedro-VirtualBox systemd-logind[573]: Watching system buttons on /dev/input/event0 (Power Button)
Nov 13 15:25:26 pedro-VirtualBox systemd-logind[573]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov 13 15:25:26 pedro-VirtualBox systemd-logind[573]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov 13 15:25:27 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 15:25:27 pedro-VirtualBox systemd-logind[573]: New session c1 of user gdm.
Nov 13 15:25:27 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 15:25:29 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.37 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 15:25:39 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov 13 15:25:39 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov 13 15:25:39 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 15:25:39 pedro-VirtualBox systemd-logind[573]: New session 2 of user pedro.
Nov 13 15:25:39 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 15:25:40 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov 13 15:25:40 pedro-VirtualBox gnome-keyring-daemon[1679]: The SSH agent was already initialized
Nov 13 15:25:40 pedro-VirtualBox gnome-keyring-daemon[1679]: The Secret Service was already initialized
Nov 13 15:25:40 pedro-VirtualBox gnome-keyring-daemon[1679]: The PKCS#11 component was already initialized
Nov 13 15:25:41 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.83 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 15:25:42 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov 13 15:25:42 pedro-VirtualBox systemd-logind[573]: Session c1 logged out. Waiting for processes to exit.
Nov 13 15:25:42 pedro-VirtualBox systemd-logind[573]: Removed session c1.
Nov 13 15:25:42 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.37, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov 13 15:25:49 pedro-VirtualBox su: (to root) pedro on pts/0
Nov 13 15:25:49 pedro-VirtualBox su: pam_unix(su-l:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 15:25:50 pedro-VirtualBox su: pam_unix(su-l:session): session closed for user root
Nov 13 15:25:53 pedro-VirtualBox dbus-daemon[557]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov 13 15:30:01 pedro-VirtualBox CRON[2458]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
Nov 13 15:30:01 pedro-VirtualBox CRON[2458]: pam_unix(cron:session): session closed for user root
Nov 13 15:34:45 pedro-VirtualBox systemd-logind[573]: System is powering down.
Nov 13 16:55:53 pedro-VirtualBox systemd-logind[661]: New seat seat0.
Nov 13 16:55:53 pedro-VirtualBox systemd-logind[661]: Watching system buttons on /dev/input/event0 (Power Button)
Nov 13 16:55:53 pedro-VirtualBox systemd-logind[661]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov 13 16:55:53 pedro-VirtualBox systemd-logind[661]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov 13 16:55:54 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 16:55:54 pedro-VirtualBox systemd-logind[661]: New session c1 of user gdm.
Nov 13 16:55:54 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 16:55:56 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.37 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 16:56:07 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:auth): authentication failure; logname= uid=0 euid=0 tty=/dev/tty1 ruser= rhost=  user=pedro
Nov 13 16:56:20 pedro-VirtualBox dbus-daemon[645]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov 13 16:56:26 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov 13 16:56:26 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov 13 16:56:26 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 16:56:26 pedro-VirtualBox systemd-logind[661]: New session 2 of user pedro.
Nov 13 16:56:26 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 16:56:27 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov 13 16:56:27 pedro-VirtualBox gnome-keyring-daemon[1769]: The SSH agent was already initialized
Nov 13 16:56:27 pedro-VirtualBox gnome-keyring-daemon[1769]: The PKCS#11 component was already initialized
Nov 13 16:56:27 pedro-VirtualBox gnome-keyring-daemon[1769]: The Secret Service was already initialized
Nov 13 16:56:28 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.83 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 16:56:29 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov 13 16:56:29 pedro-VirtualBox systemd-logind[661]: Session c1 logged out. Waiting for processes to exit.
Nov 13 16:56:29 pedro-VirtualBox systemd-logind[661]: Removed session c1.
Nov 13 16:56:29 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.37, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov 13 16:56:52 pedro-VirtualBox dbus-daemon[645]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov 13 17:07:30 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:auth): authentication failure; logname= uid=0 euid=0 tty=/dev/tty1 ruser= rhost=  user=pedro
Nov 13 17:07:34 pedro-VirtualBox gdm-password]: gkr-pam: unlocked login keyring
Nov 13 17:16:06 pedro-VirtualBox gdm-password]: gkr-pam: unlocked login keyring
Nov 13 17:16:28 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/snap/bin/code
Nov 13 17:16:28 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:16:31 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:16:44 pedro-VirtualBox dbus-daemon[645]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov 13 17:17:01 pedro-VirtualBox CRON[3540]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
Nov 13 17:17:01 pedro-VirtualBox CRON[3540]: pam_unix(cron:session): session closed for user root
Nov 13 17:17:27 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/var/lib/bind ; USER=root ; COMMAND=/usr/bin/touch db.ej10.int
Nov 13 17:17:27 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:17:27 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:17:53 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/var/lib/bind ; USER=root ; COMMAND=/usr/bin/chmod +755 db.ej10.int
Nov 13 17:17:53 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:17:53 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:18:03 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/var/lib/bind ; USER=root ; COMMAND=/usr/bin/chmod +777 db.ej10.int
Nov 13 17:18:03 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:18:03 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:19:12 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/var/lib/bind ; USER=root ; COMMAND=/usr/sbin/reboot
Nov 13 17:19:12 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:19:20 pedro-VirtualBox systemd-logind[589]: New seat seat0.
Nov 13 17:19:20 pedro-VirtualBox systemd-logind[589]: Watching system buttons on /dev/input/event0 (Power Button)
Nov 13 17:19:20 pedro-VirtualBox systemd-logind[589]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov 13 17:19:20 pedro-VirtualBox systemd-logind[589]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov 13 17:19:21 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 17:19:21 pedro-VirtualBox systemd-logind[589]: New session c1 of user gdm.
Nov 13 17:19:21 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 17:19:22 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:19:29 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov 13 17:19:29 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov 13 17:19:29 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 17:19:29 pedro-VirtualBox systemd-logind[589]: New session 2 of user pedro.
Nov 13 17:19:29 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 17:19:29 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov 13 17:19:29 pedro-VirtualBox gnome-keyring-daemon[1679]: The SSH agent was already initialized
Nov 13 17:19:29 pedro-VirtualBox gnome-keyring-daemon[1679]: The Secret Service was already initialized
Nov 13 17:19:29 pedro-VirtualBox gnome-keyring-daemon[1679]: The PKCS#11 component was already initialized
Nov 13 17:19:30 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.80 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:19:32 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov 13 17:19:32 pedro-VirtualBox systemd-logind[589]: Session c1 logged out. Waiting for processes to exit.
Nov 13 17:19:32 pedro-VirtualBox systemd-logind[589]: Removed session c1.
Nov 13 17:19:32 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov 13 17:19:47 pedro-VirtualBox dbus-daemon[568]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov 13 17:22:59 pedro-VirtualBox polkit-agent-helper-1[3243]: pam_unix(polkit-1:auth): authentication failure; logname= uid=1000 euid=0 tty= ruser=pedro rhost=  user=pedro
Nov 13 17:23:05 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:3239:21717 [/bin/sh -c cd "/home/pedro"; "/usr/bin/pkexec" --disable-internal-agent /bin/bash -c "echo SUDOPROMPT; \"/snap/code/146/usr/share/code/bin/code\" --file-write \"/home/pedro/.config/Code/code-elevated-tOrTPHu6\" \"/etc/bind/named.conf\""] (owned by unix-user:pedro)
Nov 13 17:23:05 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:23:05 pedro-VirtualBox pkexec[3240]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/bin/bash -c echo SUDOPROMPT; "/snap/code/146/usr/share/code/bin/code" --file-write "/home/pedro/.config/Code/code-elevated-tOrTPHu6" "/etc/bind/named.conf"]
Nov 13 17:23:30 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain ONE-SHOT authorization for action org.freedesktop.policykit.exec for unix-process:3276:25000 [/bin/sh -c cd "/home/pedro"; "/usr/bin/pkexec" --disable-internal-agent /bin/bash -c "echo SUDOPROMPT; \"/snap/code/146/usr/share/code/bin/code\" --file-write \"/home/pedro/.config/Code/code-elevated-moaQjcrV\" \"/etc/bind/named.conf.local\""] (owned by unix-user:pedro)
Nov 13 17:23:30 pedro-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:23:30 pedro-VirtualBox pkexec[3277]: pedro: Executing command [USER=root] [TTY=unknown] [CWD=/home/pedro] [COMMAND=/bin/bash -c echo SUDOPROMPT; "/snap/code/146/usr/share/code/bin/code" --file-write "/home/pedro/.config/Code/code-elevated-moaQjcrV" "/etc/bind/named.conf.local"]
Nov 13 17:30:01 pedro-VirtualBox CRON[4299]: pam_unix(cron:session): session opened for user root(uid=0) by (uid=0)
Nov 13 17:30:01 pedro-VirtualBox CRON[4299]: pam_unix(cron:session): session closed for user root
Nov 13 17:30:28 pedro-VirtualBox sudo: pam_unix(sudo:auth): authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/1 ruser=pedro rhost=  user=pedro
Nov 13 17:30:32 pedro-VirtualBox sudo:    pedro : TTY=pts/1 ; PWD=/etc/bind ; USER=root ; COMMAND=/usr/bin/mkdir conf
Nov 13 17:30:32 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:30:32 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:31:31 pedro-VirtualBox sudo:    pedro : TTY=pts/1 ; PWD=/etc/bind ; USER=root ; COMMAND=/usr/bin/mv named.conf named.conf.default-zones named.conf.local named.conf.options conf/
Nov 13 17:31:31 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:31:31 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:32:02 pedro-VirtualBox sudo:    pedro : TTY=pts/1 ; PWD=/etc/bind ; USER=root ; COMMAND=/usr/bin/mkdir zonas
Nov 13 17:32:02 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:32:02 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:32:47 pedro-VirtualBox sudo:    pedro : TTY=pts/1 ; PWD=/etc/bind/zonas ; USER=root ; COMMAND=/usr/bin/touch db.ej10.int
Nov 13 17:32:47 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:32:47 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:33:46 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/etc/bind/zonas ; USER=root ; COMMAND=/usr/bin/chmod 777 db.ej10.int
Nov 13 17:33:46 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:33:46 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:35:42 pedro-VirtualBox sudo:    pedro : TTY=pts/1 ; PWD=/etc/bind ; USER=root ; COMMAND=/usr/bin/rm bind.keys db.0 db.127 db.255 db.empty db.local
Nov 13 17:35:42 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:35:42 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:36:45 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/etc/bind/zonas ; USER=root ; COMMAND=/usr/sbin/reboot
Nov 13 17:36:45 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:36:45 pedro-VirtualBox sudo: pam_unix(sudo:session): session closed for user root
Nov 13 17:37:03 pedro-VirtualBox systemd-logind[665]: New seat seat0.
Nov 13 17:37:03 pedro-VirtualBox systemd-logind[665]: Watching system buttons on /dev/input/event0 (Power Button)
Nov 13 17:37:03 pedro-VirtualBox systemd-logind[665]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov 13 17:37:03 pedro-VirtualBox systemd-logind[665]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov 13 17:37:04 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 17:37:04 pedro-VirtualBox systemd-logind[665]: New session c1 of user gdm.
Nov 13 17:37:04 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 17:37:06 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.37 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:37:09 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov 13 17:37:09 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov 13 17:37:09 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 17:37:09 pedro-VirtualBox systemd-logind[665]: New session 2 of user pedro.
Nov 13 17:37:09 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 17:37:09 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov 13 17:37:10 pedro-VirtualBox gnome-keyring-daemon[1783]: The Secret Service was already initialized
Nov 13 17:37:10 pedro-VirtualBox gnome-keyring-daemon[1783]: The SSH agent was already initialized
Nov 13 17:37:10 pedro-VirtualBox gnome-keyring-daemon[1783]: The PKCS#11 component was already initialized
Nov 13 17:37:10 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.82 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:37:12 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov 13 17:37:12 pedro-VirtualBox systemd-logind[665]: Session c1 logged out. Waiting for processes to exit.
Nov 13 17:37:12 pedro-VirtualBox systemd-logind[665]: Removed session c1.
Nov 13 17:37:12 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.37, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov 13 17:37:30 pedro-VirtualBox dbus-daemon[650]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov 13 17:46:17 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-process:4474:55677 (system bus name :1.123 [/usr/bin/pkttyagent --notify-fd 5 --fallback], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:46:19 pedro-VirtualBox polkitd(authority=local): Operator of unix-session:2 successfully authenticated as unix-user:pedro to gain TEMPORARY authorization for action org.freedesktop.systemd1.manage-units for system-bus-name::1.124 [systemctl start bind9] (owned by unix-user:pedro)
Nov 13 17:46:19 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-process:4474:55677 (system bus name :1.123, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov 13 17:46:28 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/home/pedro ; USER=root ; COMMAND=/usr/sbin/reboot
Nov 13 17:46:28 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:46:46 pedro-VirtualBox systemd-logind[586]: New seat seat0.
Nov 13 17:46:46 pedro-VirtualBox systemd-logind[586]: Watching system buttons on /dev/input/event0 (Power Button)
Nov 13 17:46:46 pedro-VirtualBox systemd-logind[586]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov 13 17:46:46 pedro-VirtualBox systemd-logind[586]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov 13 17:46:47 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 17:46:47 pedro-VirtualBox systemd-logind[586]: New session c1 of user gdm.
Nov 13 17:46:47 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 17:46:48 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.36 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:46:53 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov 13 17:46:53 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov 13 17:46:53 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 17:46:53 pedro-VirtualBox systemd-logind[586]: New session 2 of user pedro.
Nov 13 17:46:53 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 17:46:53 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov 13 17:46:53 pedro-VirtualBox gnome-keyring-daemon[1761]: The SSH agent was already initialized
Nov 13 17:46:53 pedro-VirtualBox gnome-keyring-daemon[1761]: The Secret Service was already initialized
Nov 13 17:46:53 pedro-VirtualBox gnome-keyring-daemon[1761]: The PKCS#11 component was already initialized
Nov 13 17:46:54 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.81 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:46:55 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov 13 17:46:55 pedro-VirtualBox systemd-logind[586]: Session c1 logged out. Waiting for processes to exit.
Nov 13 17:46:55 pedro-VirtualBox systemd-logind[586]: Removed session c1.
Nov 13 17:46:55 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.36, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov 13 17:47:13 pedro-VirtualBox dbus-daemon[570]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov 13 17:48:03 pedro-VirtualBox dbus-daemon[570]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
Nov 13 17:48:32 pedro-VirtualBox sudo:    pedro : TTY=pts/0 ; PWD=/var/lib/bind ; USER=root ; COMMAND=/usr/sbin/reboot
Nov 13 17:48:32 pedro-VirtualBox sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Nov 13 17:48:49 pedro-VirtualBox systemd-logind[665]: New seat seat0.
Nov 13 17:48:49 pedro-VirtualBox systemd-logind[665]: Watching system buttons on /dev/input/event0 (Power Button)
Nov 13 17:48:49 pedro-VirtualBox systemd-logind[665]: Watching system buttons on /dev/input/event1 (Sleep Button)
Nov 13 17:48:49 pedro-VirtualBox systemd-logind[665]: Watching system buttons on /dev/input/event2 (AT Translated Set 2 keyboard)
Nov 13 17:48:50 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 17:48:50 pedro-VirtualBox systemd-logind[665]: New session c1 of user gdm.
Nov 13 17:48:50 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user gdm(uid=128) by (uid=0)
Nov 13 17:48:52 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:c1 (system bus name :1.39 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:49:00 pedro-VirtualBox gdm-password]: gkr-pam: unable to locate daemon control file
Nov 13 17:49:00 pedro-VirtualBox gdm-password]: gkr-pam: stashed password to try later in open session
Nov 13 17:49:00 pedro-VirtualBox gdm-password]: pam_unix(gdm-password:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 17:49:00 pedro-VirtualBox systemd-logind[665]: New session 2 of user pedro.
Nov 13 17:49:00 pedro-VirtualBox systemd: pam_unix(systemd-user:session): session opened for user pedro(uid=1000) by (uid=0)
Nov 13 17:49:00 pedro-VirtualBox gdm-password]: gkr-pam: gnome-keyring-daemon started properly and unlocked keyring
Nov 13 17:49:00 pedro-VirtualBox gnome-keyring-daemon[1848]: The SSH agent was already initialized
Nov 13 17:49:00 pedro-VirtualBox gnome-keyring-daemon[1848]: The PKCS#11 component was already initialized
Nov 13 17:49:00 pedro-VirtualBox gnome-keyring-daemon[1848]: The Secret Service was already initialized
Nov 13 17:49:01 pedro-VirtualBox polkitd(authority=local): Registered Authentication Agent for unix-session:2 (system bus name :1.81 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8)
Nov 13 17:49:02 pedro-VirtualBox gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Nov 13 17:49:02 pedro-VirtualBox systemd-logind[665]: Session c1 logged out. Waiting for processes to exit.
Nov 13 17:49:02 pedro-VirtualBox systemd-logind[665]: Removed session c1.
Nov 13 17:49:02 pedro-VirtualBox polkitd(authority=local): Unregistered Authentication Agent for unix-session:c1 (system bus name :1.39, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale es_ES.UTF-8) (disconnected from bus)
Nov 13 17:49:16 pedro-VirtualBox dbus-daemon[643]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)

