**Guia intalación de odoo con apache2 en ubuntu**
Actualizar a la última versión disponible
`sudo apt update && sudo apt upgrade -y`
Intalar herramientas que vamos a utilizar
`sudo apt install git wget curl nano unzip software-properties-common build-essential postgresql -y`
Iniciamos el servicio de sql
`sudo service postgresql start`
Creamos un usuario para posgradeSQL
`sudo su - postgres -c "createuser -s odoo17"`
Dependencias necesarias (se pueden intalar todo junto en el 1ºpaso si te es mas comodo)
`sudo apt install python3 python3-pip python3-dev \
  libxml2-dev libxslt1-dev zlib1g-dev libldap2-dev libsasl2-dev \
  libjpeg-dev libpq-dev libffi-dev libssl-dev -y
`
Creamos un usuario de sistema para odoo
`sudo adduser --system --group --home /opt/odoo odoo`
Bajamos la version 17 de odoo
`sudo git clone https://github.com/odoo/odoo --depth 1 --branch 17.0 /opt/odoo/17.0`
Instalamos el soporte vent de python 
`sudo apt install python3-venv python3-full -y`
Y creamos un entorno virtual para odoo
`python3 -m venv /opt/odoo/venv`
Nos aseguramos de que el entorno pertenece al usuario y le damos permiso de escritura y ejecución
`sudo chown -R $USER:$USER /opt/odoo
sudo chmod -R a+rX /opt/odoo/venv`
Activamos el entorno
`source /opt/odoo/venv/bin/activate`
Instalamos los requerimentos dentro del entorno ya que en caso contrario dara error
`pip install -r /opt/odoo/17.0/requirements.txt`
Creamos el servicio systemd para que use el vent
`sudo nano /etc/systemd/system/odoo.service`
*Añadimos esto:*
`[Unit]
Description=Odoo Service
After=postgresql.service

[Service]
Type=simple
User=odoo
Group=odoo
ExecStart=/opt/odoo/venv/bin/python3 /opt/odoo/17.0/odoo-bin -c /etc/odoo.conf
Restart=on-failure

[Install]
WantedBy=multi-user.target`
Y reiniciamos el servicio
`sudo systemctl daemon-reload
sudo systemctl restart odoo`
Creamos el archivo de configuración
`sudo nano /etc/odoo.conf`
*Le añadimos esto:*
`[options]
admin_passwd = admin
db_host = False
db_port = False
db_user = odoo17
db_password = False
addons_path = /opt/odoo/17.0/addons
logfile = /var/log/odoo.log
`
Y le damos permisos
`sudo chown odoo:odoo /etc/odoo.conf
sudo chmod 640 /etc/odoo.conf`
Activamos el servicio
`sudo systemctl daemon-reload
sudo systemctl enable odoo
sudo systemctl start odoo`
Ahora instalamos apache2
`sudo apt install apache2 -y`
Activamos los modulos
`sudo a2enmod proxy proxy_http headers rewrite`
Creamos el config del VirtualHost
`sudo nano /etc/apache2/sites-available/odoo.conf`
*Le añadimos esto:*
`<VirtualHost *:80>
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
`
Activamos el sitio:
`sudo a2ensite odoo.conf
sudo systemctl reload apache2`
Este es un paso opcional que no he usado pero recomendable si se usa para una VPS real para agregar seguridad
`sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache
`
