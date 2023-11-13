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

* EN EL DOCKER COMPOSE PONDREMOS *:

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