![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)

# flask-uwsgi-nginx-hola-mundo
El objetivo es publicar una aplicación **Flask** publicada en la raiz (**/**), servida por **uWSGI** como servicio y expuesta al usuario con **nginx** que actuará como **proxy inverso**.

Se describen los pasos seguidos en un **Ubuntu 20.04**, aunque el procedimiento, seguramente, es aplicable en otras distribuciones con alguna pequeña variación.

## Instalación de uWSGI.
**uWSGI** es un **servidor de aplicaciones** que puede peden estar desarrolladas en diferentes lenguajes, entre los que se encuentra **python** y **Flask** como caso particular.
Se puede instalar como paquete **pip**, pero la mayoría de distribuciones tienen una versión instalabe desde sus repositorios. Me he decantado por esta última opción.

```bash
sudo apt install uwsgi-core uwsgi-plugin-python3
```
## Creación del proyecto Falsk.
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
