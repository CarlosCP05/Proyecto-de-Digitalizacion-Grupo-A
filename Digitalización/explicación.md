
## 1. Preparación del Sistema

Actualiza los repositorios para garantizar que todo el software sea reciente:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Instalación y Configuración de Apache2

Instala el servidor web que actuará como puerta de entrada (Proxy) para Odoo:

```bash
sudo apt install apache2 -y
```

Configura el Firewall (UFW):

```bash
sudo ufw allow 'Apache'
sudo ufw reload
```

Verifica el estado de Apache:

```bash
sudo systemctl status apache2
```

---

## 3. Instalación de Dependencias (Python y PostgreSQL)

Odoo requiere una base de datos y librerías específicas de Python. Instala los paquetes del sistema:

```bash
sudo apt install postgresql postgresql-client python3-dev python3-pip python3-venv python3-wheel build-essential libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev libldap2-dev libssl-dev libffi-dev libjpeg-dev libpq-dev liblcms2-dev libblas-dev libatlas-base-dev git -y
```

Crea un usuario en PostgreSQL llamado `odoo`:

```bash
sudo su - postgres -c "createuser -s odoo"
```

---

## 4. Creación del Usuario de Sistema y Permisos

Crea el usuario `odoo`:

```bash
sudo useradd -m -d /opt/odoo -U -r -s /bin/bash odoo
```

Crea el directorio y asigna permisos:

```bash
sudo mkdir -p /opt/odoo
sudo chown -R odoo:odoo /opt/odoo
```

---

## 5. Instalación de Odoo

> 🔴 **ATENCIÓN:** En esta sección NO uses `sudo` directamente. Debes trabajar como el usuario `odoo`.

Cambia al usuario `odoo`:

```bash
sudo su - odoo
# El prompt de tu terminal debe decir ahora odoo@...
```

Descarga Odoo 18:

```bash
git clone https://github.com/odoo/odoo.git --depth 1 --branch 18.0 /opt/odoo/odoo-server
```

Crea y activa el entorno virtual:

```bash
cd /opt/odoo
python3 -m venv odoo-venv
source odoo-venv/bin/activate
```

Instala dependencias de Python:

```bash
pip3 install wheel
pip3 install -r odoo-server/requirements.txt
```

Sal del usuario `odoo` una vez terminada la instalación de requisitos:

```bash
deactivate
exit
# Verifica que has vuelto a tu usuario normal con permisos sudo
```

---

## 6. Configuración de Odoo (odoo.conf)

Crea el archivo de configuración del servidor Odoo:

```bash
sudo nano /etc/odoo.conf
```

Pega el siguiente contenido (cambia `tu_contraseña_maestra` por una segura):

```ini
[options]
admin_passwd = tu_contraseña_maestra
db_host = False
db_port = False
db_user = odoo
db_password = False
addons_path = /opt/odoo/odoo-server/addons
proxy_mode = True
xmlrpc_port = 8069
```

Protege el archivo:

```bash
sudo chown odoo: /etc/odoo.conf
sudo chmod 640 /etc/odoo.conf
```

---

## 7. Crear Servicio Systemd

Para que Odoo se inicie automáticamente al encender el servidor, crea el archivo de servicio:

```bash
sudo nano /etc/systemd/system/odoo.service
```

Pega el siguiente contenido:

```ini
[Unit]
Description=Odoo
Documentation=https://www.odoo.com

[Service]
# Usuario y Grupo que ejecutan Odoo
User=odoo
Group=odoo
ExecStart=/opt/odoo/odoo-venv/bin/python3 /opt/odoo/odoo-server/odoo-bin -c /etc/odoo.conf
Restart=on-failure

[Install]
WantedBy=default.target
```

Activa e inicia Odoo:

```bash
sudo systemctl daemon-reload
sudo systemctl start odoo
sudo systemctl enable odoo
```

Verifica que corre:

```bash
sudo systemctl status odoo
```

---

## 8. Configurar Proxy Inverso (Apache)

Conecta el puerto 80 (Apache) con el puerto 8069 (Odoo).

Habilita los módulos necesarios:

```bash
sudo a2enmod proxy proxy_http headers rewrite
sudo systemctl restart apache2
```

Crea el archivo de host virtual:

```bash
sudo nano /etc/apache2/sites-available/odoo.conf
```

Pega la siguiente configuración (ajusta el dominio si lo tienes):

```apache
<VirtualHost *:80>
    # Pon tu dominio aquí si tienes, si no, déjalo así o usa la IP
    ServerName midominio.com
    ServerAlias www.midominio.com

    ErrorLog ${APACHE_LOG_DIR}/odoo_error.log
    CustomLog ${APACHE_LOG_DIR}/odoo_access.log combined

    ProxyPreserveHost On
    ProxyRequests Off

    # Redirección al puerto de Odoo
    ProxyPass / http://127.0.0.1:8069/
    ProxyPassReverse / http://127.0.0.1:8069/

    # Cabeceras
    RequestHeader set "X-Forwarded-Proto" "http"
</VirtualHost>
```

Activa el sitio y recarga Apache:

```bash
sudo a2ensite odoo.conf
sudo systemctl reload apache2
```

---

## 9. Acceso Final

Abre tu navegador web e ingresa a:

```
http://TU_IP:EL_PUERTO
# o tu dominio
```

Verás la pantalla de configuración de base de datos.

- **Master Password:** Usa la contraseña que definiste en el paso 6 (`admin_passwd`).
- Rellena los datos (Email, Password, Idioma) y pulsa **Create Database**.

---