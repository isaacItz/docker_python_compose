# DOCKER PRACTICA 2

## ¿Qué es es Flask?
Vamos a crear un aplicación en python y la vamos a guardarla en un contenedor. Esta aplicación utiliza flask. Flask es un "micro" Framework escrito en Python y concebido para facilitar el desarrollo de Aplicaciones Web bajo el patrón MVC.

La palabra "micro" no designa a que sea un proyecto pequeño o que nos permita hacer páginas web pequeñas sino que al instalar Flask tenemos las herramientas necesarias para crear una aplicación web funcional pero si se necesita en algún momento una nueva funcionalidad hay un conjunto muy grande extensiones (plugins) que se pueden instalar con Flask que le van dotando de funcionalidad.

El patrón MVC es una manera o una forma de trabajar que permite diferenciar y separar lo que es el modelo de datos (los datos que van a tener la App que normalmente están guardados en BD), la vista (página HTML) y el controlador (donde se gestiona las peticiones de la app web).

Las principales ventajas de Flask son:
1) Flask es un "micro" Framework: para desarrollar una App básica o que se quiera desarrollar de una forma ágil y rápida Flask puede ser muy conveniente, para determinadas aplicaciones no se necesitan muchas extensiones y es suficiente
2) Incluye un servidor web de desarrollo: no se necesita una infraestructura con un servidor web para probar las aplicaciones sino de una manera sencilla se puede correr un servidor web para ir viendo los resultados que se van obteniendo
3) Tiene un depurador y soporte integrado para pruebas unitarias: si tenemos algún error en el código que se está construyendo se puede depurar ese error y se puede ver los valores de las variables. Además está la posibilidad de integrar pruebas unitarias
4) Es compatible con Python3
5) Es compatible con wsgi: Wsig es un protocolo que utiliza los servidores web para servir las páginas web escritas en Python
6) Buen manejo de rutas: cuando se trabaja con Apps Web hechas en Python se tiene el controlador que recibe todas las peticiones que hacen los clientes y se tienen que determinar que ruta está accediendo el cliente para ejecutar el código necesario
7) Soporta de manera nativa el uso de cookies seguras
8) Se pueden usar sesiones
9) Flask no tiene ORMs: pero se puede usar una extensión
10) Sirve para construir servicios web (como APIs REST) o aplicaciones de contenido estático.
Flask es Open Source y está amparado bajo una licencia BSD
12) Buena documentación, código de GitHub y lista de correos

Flask tambien tiene diferentes extensiones para ser usadas. Algunas comunes:
* flask-script: Permite tener un comando de la línea de comando para manejar la aplicación.
* flask-Bootstrap: Hojas de estilo para la página
* flask-WTF: Sirve para generar formularios de HTML con clases y objetos
* flask-Sqlalchemy: Sirve para poder generar el modelo de datos
* flask-login: Sirve para la autenticación de usuario y contraseña

## Contenedor básico de Python/Flask
1) Crear un nuevo repositorio con visibilidad pública en GitHub
* Nombre: `docker_python`
* Descripción: `Ejemplo para contenedor Docker Python`
2) Copiar los archivos desde la URL https://github.com/Juli-BCN/docker_python
* Cada uno puede utilizar el método preferido para copiar como hacerlo a mano, usar un clon en Visual Studio (git clone + git push) o hacer un "Fork" desde GitHub
3) Existe un archivo llamado `app.py` con el siguiente contenido:
```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(),  visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```
4) Nuestra aplicación tiene una serie de dependencias (librerías de terceros) que guardaremos en el archivo `reqs.txt`, con el siguiente contenido:
```
Flask
Redis
```
3) Por último, vamos a modificar el archivo `Dockerfile` para personalizar el contenedor:
```
# Partimos de una base oficial de python
# FROM python:2.7-slim
FROM python:3.8-slim-buster

# El directorio de trabajo es desde donde se ejecuta el contenedor al iniciarse
WORKDIR /app

# Copiamos todos los archivos del build context al directorio /app del contenedor
COPY . /app

# Ejecutamos pip para instalar las dependencias en el contenedor
RUN pip install --trusted-host pypi.python.org -r reqs.txt

# Indicamos que este contenedor se comunica por el puerto 80/tcp
EXPOSE 80

# Declaramos una variable de entorno
ENV NAME Python_App

# Ejecuta nuestra aplicación cuando se inicia el contenedor
CMD ["python", "app.py"]
```
4) Modificar las línea 4 para agregar los valores personales de cada uno:
```
LABEL maintainer="JuliBCN <julibcn@gmail.com>"
```


## Creación del contenedor
Para crear un contenedor desde un archivo `Dockerfile` usaremos el comando `docker build` con el `-t` para agregar una etiqueta:
> docker build -t python_docker:v1 .

Y podremos ver que una nueva imagen está instalada en nuestra sesión con el comando `docker images`:
> docker images

Podemos utlizar mas comandos para ver imagenes, como `docker image ls`, hay posibilidades con el mismo efecto:
> docker image ls


## Ejecución del contenedor Python-Flask
amos a arrancar nuestro contenedor y probar la aplicación:
* -d - detached, para que podamos utilizar el mismo shell de comandos
* -p - para especificar los puertos que se usaran tanto en el host como en el contenedor
* --rm - como a veces tenemos contenedores de prueba, esto borrara el contenedor una vez se pare

Para ejecutarlo, escribiremos:
> docker run -d --rm -p44444:80 python_docker:v1

Lo que arranca la aplicación Flask en el puerto 44444 del host pero enmascarado en TCP/443 (HTTPS). Debemos navegar al resultado y veremos algo parecido a esto:
```
Hello Python_App!

Hostname: 8425ca419a97
Visits: cannot connect to Redis, counter disabled
```

![python_app1.png](./python_app1.png)


## Parada del contenedor Python-Flask
Para parar un contenedor en ejecución, primero hemos de saber el ID de dicho contenedor con el comando `docker ps`. Una vez averiguado, podemos pararlo con `docker stop`:
> docker ps

![container_id.png](./container_id.png)

> docker stop XXXXXXXXXXXX


## Docker Compose
En este caso, nuestro contenedor no funciona por sí mismo. Es muy habitual que dependamos de servicios para poder iniciar la aplicación, habitualmente bases de datos. En este caso necesitamos una base de datos Redis que no tenemos. Docker Compose es una herramienta que permite simplificar el uso de Docker. A partir de archivos YAML es mas sencillo crear contendores, conectarlos, habilitar puertos, volúmenes, etc.

Con Compose podemos crear diferentes contenedores y al mismo tiempo, en cada contenedor, diferentes servicios, unirlos a un volumen común, iniciarlos y apagarlos, etc. Es un componente fundamental para poder construir aplicaciones y microservicios.

Docker Compose, por defecto, busca instrucciones en el archivo `docker-compose.yml`. La principal diferencia con respecto al ejemplo anterior, es que en un servicio podemos indicar una imagen (parámetro `image`) o un contexto de configuración (parámetro `build`). Esta es una manera de integrar las dos herramientas que nos proporciona Docker: la creación de imágenes y la composición de aplicaciones con servicios.
