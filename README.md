![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)

# flask-uwsgi-nginx-hola-mundo
El objetivo es publicar una aplicación **Flask** publicada en la raiz (**/**), servida por **uWSGI** como servicio y expuesta al usuario con **nginx** que actuará como **proxy inverso**.

Se describen los pasos seguidos en un **Ubuntu 20.04**, aunque el procedimiento, seguramente, es aplicable en otras distribuciones con alguna pequeñ variación.

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
