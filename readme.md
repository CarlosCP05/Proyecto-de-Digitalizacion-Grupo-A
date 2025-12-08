**Guía instalación de Odoo 17 con Apache2 en Ubuntu/WSL**

Actualizar a la última versión:

```bash
sudo apt update && sudo apt upgrade -y
```

Instalar herramientas y dependencias necesarias:

```bash
sudo apt install git wget curl nano unzip software-properties-common build-essential postgresql python3 python3-pip python3-dev \
libxml2-dev libxslt1-dev zlib1g-dev libldap2-dev libsasl2-dev libjpeg-dev libpq-dev libffi-dev libssl-dev python3-venv python3-full apache2 -y
```

Iniciamos el servicio de PostgreSQL:

```bash
sudo service postgresql start
```

Creamos un usuario de PostgreSQL (recomendado usar el mismo que el usuario de Linux):

```bash
sudo -u postgres createuser -s $USER
```

> Si prefieres usar contraseña:
>
> ```bash
> sudo -u postgres psql
> ALTER USER $USER WITH PASSWORD 'tu_contraseña';
> \q
> ```

Creamos usuario de sistema para Odoo:

```bash
sudo adduser --system --group --home /opt/odoo odoo
```

Descargamos Odoo 17:

```bash
sudo git clone https://github.com/odoo/odoo --depth 1 --branch 17.0 /opt/odoo/17.0
```

Creamos el entorno virtual y lo configuramos:

```bash
python3 -m venv /opt/odoo/venv
sudo chown -R $USER:$USER /opt/odoo
sudo chmod -R a+rX /opt/odoo/venv
source /opt/odoo/venv/bin/activate
pip install --upgrade pip setuptools wheel
pip install -r /opt/odoo/17.0/requirements.txt
```

Creamos el archivo de configuración en nuestro home:

```bash
nano ~/odoo.conf
```

Contenido:

```
[options]
admin_passwd = admin
db_host = False
db_port = False
db_user = $USER
db_password = False # o 'tu_contraseña' si configuraste
addons_path = /opt/odoo/17.0/addons
logfile = /home/$USER/odoo.log
```

```bash
chmod 600 ~/odoo.conf
```

Iniciamos Odoo dentro del entorno virtual:

```bash
cd /opt/odoo/17.0
source /opt/odoo/venv/bin/activate
./odoo-bin -c ~/odoo.conf
```

> Opcional: ejecutarlo en segundo plano

```bash
nohup ./odoo-bin -c ~/odoo.conf &
```

Instalamos Apache2 y activamos módulos:

```bash
sudo a2enmod proxy proxy_http headers rewrite
```

Creamos la configuración del VirtualHost:

```bash
sudo nano /etc/apache2/sites-available/odoo.conf
```

Contenido:

```
<VirtualHost *:80>
    ServerName tu-dominio.com

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8069/
    ProxyPassReverse / http://127.0.0.1:8069/

    <Location />
        Require all granted
    </Location>

    ErrorLog ${APACHE_LOG_DIR}/odoo-error.log
    CustomLog ${APACHE_LOG_DIR}/odoo-access.log combined
</VirtualHost>
```

Activamos el sitio y recargamos Apache:

```bash
sudo a2ensite odoo.conf
sudo systemctl reload apache2
```

**Notas importantes:**


* Los logs deben estar en un directorio escribible (`~/odoo.log`).
* Si configuras contraseña en PostgreSQL, actualiza `db_password` en `odoo.conf`.
* Odoo estará disponible en `http://localhost:8069`.
