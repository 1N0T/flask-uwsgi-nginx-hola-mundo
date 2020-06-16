![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)

# flask-uwsgi-nginx-hola-mundo
El objetivo es publicar una aplicación **Flask** expuesta a nivel de la raiz (**/**), servida por **uWSGI** como servicio y expuesta al usuario con **nginx** que actuará como **proxy inverso**.

Se describen los pasos seguidos en un **Ubuntu 20.04**, aunque el procedimiento, seguramente, es aplicable en otras distribuciones con alguna pequeña variación.

## Instalación de uWSGI.
**uWSGI** es un **servidor de aplicaciones** que puede peden estar desarrolladas en diferentes lenguajes, entre los que se encuentra **python** y **Flask** como caso particular.
Se puede instalar como paquete **pip**, pero la mayoría de distribuciones tienen una versión instalabe desde sus repositorios. Me he decantado por esta última opción.

```bash
sudo apt install uwsgi-core uwsgi-plugin-python3
```
## Creación del proyecto Flask.
Creamos el directorio del proyecto, entorno virtual y una aplicación Flask mínima de ejemplo.
```bash
mkdir flaskuwsgi
cd flaskuwsgi
python3 -m venv venv
source venv/bin/activate
pip3 install flask
```
## Fuentes aplicación Flask.
Crearemos una aplicación que se servirá como **http://localhost:5000/** que, cuando lleguemos al final, se servirá como **http://localhost/app**. Para contemplar las situaciones más habituales, crearé dos fuentes a diferente nivel en la estructura de directorios y con referencias entre ellos para validar que los enlaces entre páginas se redirigen correctamente.
```bash
nano app.py
```
```python
from flask import Flask
from api.xxx import api_routes

APP = "/"
mi_app = Flask(__name__)

mi_app.register_blueprint(api_routes, url_prefix=APP)

@mi_app.route(APP)
def index():
    return "<a href='api/xxx'>api</a><br/><span style='color:red'>uWSGI parece estar funcionando</span>"

if __name__ == "__main__":
    mi_app.run(host='0.0.0.0')

```
```bash
mkdir api
nano api/xxx.py
```
```python
from flask import Blueprint

api_routes = Blueprint("api_routes", __name__)

@api_routes.route("/api/xxx")
def index_xxx():
    return "<a href='/'>home</a><br/><span style='color:red'>uWSGI parece estar funcionando api/xxx</span>"
```
Nada del otro mundo, pero suficiente para probar que todo funciona según lo deseado. Aprovechamos para crear los ficheros **__init__.py**
```bahs
touch __init__.py
touch api/__init__.py
```
LLegado a este punto, ya tenemos una aplicación operativa que podemos probar en http://localhost:5000 después de ejecutar lo siguiente:
```bash
python3 app.py
```
## Servir aplicación Flask con uWSGI.
Una vez comprobado que la aplicación anteriro es funcional, la vamos a publicar utilizando **uWSGI** como servidor de aplicación.

Como primer paso, vamos a publicar la misma aplicación utilizando **uWSGI** utilizando el protocolo **http**, lo que nos permitirá seguir utilizando el navegador para probar nuestra maravillosa aplicación.
```bash
nano uwsgi.ini
```
```ini
[uwsgi]

socket = 0.0.0.0:5000
protocol = http
chdir = /home/user/flaskuwsgi/
venv = ./venv/
plugin = python3
module = app:mi_app
master = true
processes = 5

socket = flaskuwsgi.sock
chmod-socket = 666
vacuum = true

uid = nobody
gid = nogroup

die-on-term = true
```
Si ejecutamos el siguiente comando, será uWSGI quien sirva la aplicación anterior actuando como un servidor web al uso.
```bash
uwsgi /home/user/flaskuwsgi/uwsgi.ini
```
Llegado a este punto, todo debería seguir funcionando exactamente igual que antes. Comprobado ésto, podemos comentar la línea **protocol = http** anteponiendo una **#**. Esto hará que se sirva un **socket unix** en lugar de **http**. 

Acto seguido, comprobaremos que la aplicación ya no es usable desde un navegador, por lo que tendremos que anteponer un servidor web que haga de proxy. Utilizaremos **nginx** que gispone de un módulo para comunicar con **uWSGI** de forma nativa.

En lugar de un fichero de configuración, también es posible pasar los valores como parámetros (o mezclar ambos).
```bash
uwsgi --http-socket 127.0.0.1:6006 --plugin python3    \
      --chdir /home/tonis/sources/python/flaskuwsgi/   \
      --venv ./venv/ --module app --callable mi_app    \
      --processes 4 --threads 2 --stats 127.0.0.1:9191 
```
La opción **--stats** publica consultar información sobre el estado del servidor.

## Instalación niginx.
```bash
sudo apt install nginx
cd /etc/nginx/
sudo nano /etc/nginx/sites-available/faskuwsgi
```
```nginx
server {
    listen 80;
    server_name localhost;
    # Descomentar si queremos ver los rewrites aplicados
    # error_log /var/log/nginx/error.log notice;
    # rewrite_log on;

    location /app {
    
        # Indicamos que tipo de contenido queremos inspeccionar para modificar
        sub_filter_types *;
        sub_filter_once off;
        sub_filter_last_modified on;
        
        # Tenemos que asegurarnos que las URLs que existen dentro del contenido se reescriben
        # para que apunten donde queremos. Todo lo que sea /..., lo convertimos a /app/...
        sub_filter 'href="/' 'href="/app/';
        sub_filter "href='/" "href='/app/";

        # combinamos la acción de proxy inverso y de proxy uwsgi
        proxy_pass http://127.0.0.1:5000/;
        rewrite /app(.*) $1 break;
        include uwsgi_params;
        uwsgi_pass unix:/home/user/flaskuwsgi/flaskuwsgi.sock;
    }
}
```
```bash
sudo ln -s /etc/nginx/sites-available/faskuwsgi /etc/nginx/sites-enabled
sudo systemctl reload nginx
uwsgi /home/user/flaskuwsgi/uwsgi.ini
```
Antes debemeos asegurarnos de deshabilitar el protocolo http en el fichero de configuración. Así utilizaremos la comunicación nativa entre nginx y uWSGI.

Si no nos hemos equivocado, podremos acceder como hasta ahora, per en http://localhost/app y todo debería seguir funcionando igual.

## uWSGI como servicio.
Para que uWSGI publique nuestra aplicación cada vez que iniciamos el servidor, podemos colocrlo en el fichero **/etc/rc.local** o, si queremos ser más elegantes, configurarlo como servicio. Estos son los pasos a seguir.

```bash
sudo nano /etc/systemd/system/flaskuwsgi.service 
```
```ini
Description=Flask uWSGI ejemplo
After=network.target

[Service]
User=tonis
Group=tonis
WorkingDirectory=/home/user/flaskuwsgi
Environment="PATH=/home/user/flaskuwsgi/venv/bin"
ExecStart=/usr/bin/uwsgi uwsgi.ini

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl start flaskuwsgi
sudo systemctl enable flaskuwsgi

```
