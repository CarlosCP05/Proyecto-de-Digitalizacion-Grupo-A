# Tutorial de instalación de Odoo

## Aparte de necesitar python instalado, Odoo utiliza PostgreSQL como base de datos.
1. Descarga PostgreSQL desde su web oficial.
2. Instálalo y elige una contraseña para el usuario “postgres”.

## Es recomendable mantener Odoo aislado del resto del sistema.
1. Crea una carpeta para tu proyecto
2. Crea el entorno virtual:
```python -m venv venv```
3. Activa el entorno virtual:
```venv\Scripts\activate```

## Descargar el código de Odoo
Clonar o descargar el archivo zip del repositorio git de odoo:
```git clone https://github.com/odoo/odoo.git```

## Instalar dependencias de Python
1. Desde el directorio de odoo usar el siguiente comando:
   ```pip install -r requirements.txt```
2. Si no funciona, primero se debe instalar:
   ```pip install wheel setuptools```

## Crear archivo de configuraciones
El archivo de debe llamar "odoo.conf" y debe incluir: 
```
[options]
addons_path = addons
db_user = postgres
db_password = CONTRASEÑA
xmlrpc_port = 8080
```

## Crear una base de datos en PostgreSQL
Abrir PostgreSQL desde la consola y crear un usuario con una base de datos
```
psql -U postgres
CREATE USER odoo WITH PASSWORD 'odoo';
ALTER ROLE odoo WITH SUPERUSER;
```

## Ejecutar Odoo
Desde el directorio de odoo otra vez, ejecutar el siguiente comando: 
```
python odoo-bin -c odoo.conf
```
Y ya se puede comprobar si funciona en http://localhost:8080
