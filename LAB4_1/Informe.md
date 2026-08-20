# Laboratorio 4.1: Instancias Computacionales, Acceso Seguro y Bases de Datos Gestionadas (EC2 y RDS)

## Ejercicio 3.1: Lanzamiento y acceso a una instancia EC2
![](images/ec2_panel.png)
![](images/ec2_conf1.png)
![](images/ec2_conf2.png)
![](images/ec2_conf3.png)
Para iniciar el entorno de cómputo en la nube, se realizó el despliegue de una instancia Amazon EC2 utilizando una AMI de Ubuntu Server LTS, seleccionando una clase t3.micro para garantizar la optimización de recursos bajo el marco de la capa gratuita de AWS.
![](images/key1.png)
Durante el proceso de inicialización, se llevó a cabo la creación de un Par de Claves (Key Pair) para asegurar el acceso remoto a la instancia. Tras descargar el archivo privado .pem, este fue almacenado de forma segura fuera del repositorio del proyecto para evitar exposiciones accidentales de credenciales.
![](images/grupo_seg.png)
![](images/ec2_conf4.png)
![](images/ec2_resumen.png)
![](images/ec2_listo.png)
Con eso la instnacia ya estaría lista para un lanzamiento, pero le añadiremos configuraciones de seguridad como lo son los grupos de seguridad
![](images/ssh1.png)
![](images/ssh2.png)
![](images/ssh3.png)
![](images/ssh4.png)
![](images/ssh5.png)
![](images/ssh6.png)
Esta configuración establece la base de seguridad para la administración de la infraestructura, garantizando que solo el titular de la clave privada pueda establecer una sesión de terminal segura hacia la instancia.

## Ejercicio 3.2: Configuración del grupo de seguridad para HTTP y HTTPS
![](images/ec2_grupo1.png)
![](images/ec2_grupo2.png)
![](images/ec2_grupo3.png)
![](images/ec2_grupo4.png)
![](images/ec2_grupo5.png)
Para permitir la correcta administración del servidor y la futura publicación de los servicios web, se completó la configuración del grupo de seguridad ec2-api-sg estableciendo las reglas de tráfico de entrada y salida necesarias
Esta estructura asegura que los servicios públicos operen de forma correcta mientras que los puertos internos y de bases de datos permanecen protegidos frente a accesos externos no autorizados
```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
curl -I http://localhost
```
## Ejercicio 3.3: Creación de RDS y comunicación segura con EC2
Para la persistencia de datos del proyecto, se aprovisionó una instancia de Amazon RDS utilizando el motor MariaDB (o MySQL), seleccionando una clase compatible con la capa gratuita de AWS para asegurar la eficiencia de costos. La base de datos fue configurada bajo el nombre db_apimovies
![](images/rds1.png)
![](images/rds2.png)
- Aislamiento de Red: La instancia RDS se desplegó en la misma VPC que la instancia EC2, garantizando una comunicación interna y de baja latencia.

- Acceso Restringido: Se desactivó la opción de Acceso público en la configuración de RDS, cumpliendo con el estándar de seguridad de no exponer la base de datos directamente a Internet.

- Grupo de Seguridad Dedicado (rds-movies-sg): Se creó un grupo de seguridad exclusivo para la base de datos con una política de acceso de "mínimo privilegio". La única regla de entrada permitida es la conexión al puerto 3306 (MySQL/Aurora), la cual está restringida únicamente al ID del grupo de seguridad de la instancia EC2 (ec2-api-sg), garantizando que solo la aplicación pueda comunicarse con la base de datos.
  

![](images/rds3.png)


Una vez configurado el grupo de seguridad rds-movies-sg, este se asoció a la instancia RDS para aplicar las políticas de tráfico definidas. Es importante destacar que, bajo este modelo de seguridad, no fue necesario habilitar el puerto 3306 en el grupo de seguridad ec2-api-sg, ya que el tráfico se autoriza mediante la referencia de grupos de seguridad y no a través de reglas de entrada en la instancia de aplicación.

![](images/maria1.png)
![](images/maria2.png)
![](images/maria3.png)

```bash
sudo apt update
sudo apt install -y mariadb-client
mariadb -h ENDPOINT_RDS -P 3306 -u admin -p
```
Para validar que la comunicación entre ambos servicios fuera exitosa y privada, se procedió a instalar el cliente de MariaDB en la instancia EC2 y a establecer la conexión con el endpoint proporcionado por RDS
Tras ingresar las credenciales administrativas, se confirmó la conectividad exitosa mediante la consola SQL. Esta prueba garantiza que la instancia EC2 posee los permisos necesarios para interactuar con la base de datos dentro del entorno de red privada, cumpliendo con los requisitos de seguridad del laboratorio al evitar la exposición directa de la base de datos hacia Internet.
![](images/maria4.png)
```bash
CREATE DATABASE IF NOT EXISTS db_apimovies
  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER 'usr_movies'@'%' IDENTIFIED BY 'CAMBIAR_ESTA_CONTRASENA';
GRANT ALL PRIVILEGES ON db_apimovies.* TO 'usr_movies'@'%';
FLUSH PRIVILEGES;
```
Tras verificar la conectividad, se procedió a estructurar el entorno de datos. Se ejecutaron las sentencias SQL necesarias para crear la base de datos db_apimovies, asegurando la correcta codificación de caracteres con utf8mb4 para soportar una amplia variedad de símbolos y asegurar la integridad de la información.  Para cumplir con el principio de seguridad de no utilizar el usuario administrativo para la operación diaria de la aplicación, se creó un usuario específico (usr_movies) con acceso restringido únicamente a la base de datos necesaria, delegándole todos los privilegios requeridos para las operaciones de la API
## Ejercicio 3.4: Clonación y despliegue de la API CRUD Movies
Para llevar a cabo el despliegue del servicio web, se utilizó el proyecto api-restful-crud-movies, una API REST desarrollada con Express.js que emplea la librería mysql2 para establecer la comunicación con bases de datos MySQL o MariaDB.
![](images/git1.png)
```bash
sudo apt update
sudo apt install -y git
# Comprueba que Node.js sea 18 o superior.
node --version
npm --version
```
Como paso previo, se actualizó el sistema operativo e instalaron las herramientas necesarias para la gestión del código fuente y la ejecución del entorno de JavaScript

A continuación, se clonó el repositorio oficial del proyecto en la instancia EC2, se instalaron los módulos necesarios y se generó el archivo de configuración de variables de entorno a partir de la plantilla provista:
![](images/git2.png)

```bash
git clone https://github.com/marceloquispeortega/api-restful-crud-movies
cd api-restful-crud-movies
npm install
cp .env.example .env
nano .env
```
Tras la instalación de las dependencias, se configuró el entorno de ejecución de la API para que esta pudiese comunicarse correctamente con la instancia de RDS.
Se editó el archivo .env con las credenciales correspondientes a nuestra base de datos. Este archivo es el punto de conexión seguro que permite a la aplicación localizar el servidor de base de datos (DB_HOST) y autenticarse mediante las credenciales creadas previamente (DB_USER, DB_PASSWORD)
![](images/git3.png)
![](images/git4.png)
Para permitir el funcionamiento de las operaciones CRUD, se ejecutaron las instrucciones DDL (Data Definition Language) en el cliente de RDS para crear la estructura de la tabla movies, asegurando que cada película cuente con un identificador único y restricciones de unicidad en los títulos. Adicionalmente, se cargaron registros iniciales para validar la persistencia:
![](images/git5.png)
![](images/git6.png)
```bash
USE db_apimovies;

-- Creación de la tabla de películas
CREATE TABLE IF NOT EXISTS movies (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(150) NOT NULL UNIQUE,
  year INT
);

-- Inserción de datos iniciales de prueba
INSERT IGNORE INTO movies (title, year) VALUES
  ('Inception', 2010),
  ('The Matrix', 1999),
  ('Interstellar', 2014);
```

Una vez configuradas las variables de entorno y la base de datos, se procedió a verificar el funcionamiento inicial de la API de manera local en la instancia EC2.  
Se ejecutó la aplicación directamente con Node.js:
![](images/node1.png)
```bash
node app.js
```
Posteriormente, desde una sesión SSH secundaria en la misma instancia, se validó la respuesta de los servicios mediante una petición HTTP local:
![](images/node2.png)
```bash
curl http://localhost:3000/movies
```
Dado que la ejecución directa se detiene al cerrar la sesión de terminal, se implementó el gestor de procesos PM2 para mantener la API en ejecución de forma continua y asegurar su persistencia ante reinicios o desconexiones:
![](images/pm1.png)
![](images/pm2.png)
```bash
sudo npm install --global pm2
pm2 start app.js --name movies-api
pm2 save
```
## Ejercicio 3.5: Subdominio de Cloudflare y acceso HTTPS

Para completar la publicación de la API bajo el subdominio asignado (benjamin-pereira.rootcode.com.bo), se procedió a verificar la resolución de nombres de dominio mediante el comando nslookup
![](images/cloudfare1.png)
![](images/ip.png)
```bash
nslookup SUBDOMINIO
```
Con el fin de garantizar que la comunicación entre Cloudflare y el servidor de origen (EC2) esté completamente cifrada y sea segura, se procedió a la instalación del Certificado de Origen (Origin Certificate) proporcionado por el docente.  

Para proteger las credenciales, la transferencia desde el equipo local hacia la instancia EC2 se realizó mediante el protocolo SCP (Secure Copy Protocol), utilizando la clave privada de acceso SSH (ec2-api-ssh-key.pem) para establecer un canal seguro de transferencia.
![](images/scp.png)
```bash
scp cloudflare-origin.pem ubuntu@IP_PUBLICA:/tmp/
scp cloudflare-origin.key ubuntu@IP_PUBLICA:/tmp/
```
Una vez transferidos los archivos a la instancia EC2, se procedió a su reubicación en un directorio protegido (/etc/nginx/ssl/cloudflare/). Se utilizaron los comandos install con parámetros de permisos restringidos (644 para el certificado y 600 para la clave privada) para garantizar la integridad y confidencialidad de la clave del servidor, cumpliendo así con las mejores prácticas de seguridad en despliegues de servidores web Linux.
![](images/claves.png)
```bash
sudo mkdir -p /etc/nginx/ssl/cloudflare
sudo install -m 644 /tmp/cloudflare-origin.pem /etc/nginx/ssl/cloudflare/SUBDOMINIO.pem
sudo install -m 600 /tmp/cloudflare-origin.key /etc/nginx/ssl/cloudflare/SUBDOMINIO.key
sudo chown root:root /etc/nginx/ssl/cloudflare/SUBDOMINIO.*
```
![](images/conf0.png)
![](images/conf1.png)
![](images/conf3.png)

```bash
sudo nano /etc/nginx/sites-available/movies-api


server {
    listen 80;
    listen [::]:80;
    server_name SUBDOMINIO;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name SUBDOMINIO;

    ssl_certificate /etc/nginx/ssl/cloudflare/SUBDOMINIO.pem;
    ssl_certificate_key /etc/nginx/ssl/cloudflare/SUBDOMINIO.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

sudo ln -s /etc/nginx/sites-available/movies-api /etc/nginx/sites-enabled/movies-api
sudo nginx -t
sudo systemctl reload nginx
curl -i https://SUBDOMINIO/movies
```

Con el fin de enrutar las peticiones públicas HTTPS hacia la aplicación de Node.js que opera localmente en el puerto 3001 (gestionada mediante el administrador de procesos PM2), se configuró Nginx creando el archivo virtual host /etc/nginx/sites-available/movies-api.

La configuración implementada consta de dos bloques principales:

Redirección HTTP a HTTPS: Un bloque que escucha en el puerto 80 y redirige de manera permanente (código 301) todo el tráfico inseguro hacia el protocolo cifrado HTTPS.

Gestión SSL/TLS y Proxy Pass: Un bloque que escucha en el puerto 443 con SSL habilitado, utilizando los certificados de Cloudflare y estableciendo la directiva proxy_pass hacia [http://127.0.0.1:3001](http://127.0.0.1:3001), reenviando cabeceras esenciales como el host real y las IPs de origen.

Tras activar el sitio mediante un enlace simbólico en /etc/nginx/sites-enabled/ y validar la sintaxis con sudo nginx -t, se recargó el servicio con sudo systemctl reload nginx

Para comprobar el funcionamiento integral del despliegue, se ejecutó una petición HTTP de prueba utilizando la herramienta curl desde la propia instancia y validando el acceso a través del navegador web:

![](images/result.png)

# 4. Práctica Individual 
## 4.1. Introducción a la Práctica Individual
Como parte de la ampliación de la arquitectura desplegada, se procedió a verificar y validar el funcionamiento integral de todas las operaciones del ciclo de vida de los datos (CRUD: Crear, Leer, Actualizar y Eliminar). La aplicación, que opera sobre la instancia AWS EC2 gestionada mediante PM2 y expuesta de forma segura mediante Nginx y Cloudflare, fue sometida a pruebas funcionales utilizando la herramienta Postman para comprobar tanto la persistencia de los registros en la base de datos MariaDB alojada en Amazon RDS como la efectividad de las reglas de validación implementadas en la API.
## 4.2. Evidencias de Seguridad y Conectividad en AWS
Para cumplir con los requisitos de aislamiento y protección de la infraestructura, se documentaron los siguientes elementos de red:

- Reglas de Entrada del Grupo de Seguridad de EC2: Se configuró el acceso restringido permitiendo únicamente los puertos esenciales: SSH (22), HTTP (80) y HTTPS (443) desde las fuentes autorizadas, manteniendo el tráfico de salida abierto para la operatividad del sistema.

![](images/reglas.png)

- Aislamiento del Grupo de Seguridad de RDS (Puerto 3306): Se restringió el acceso a la base de datos MariaDB para que el puerto 3306 sea accesible únicamente desde el Grupo de Seguridad de la instancia EC2, evitando cualquier exposición pública directa (0.0.0.0/0).

![](images/reglas2.png)

- Conexión Exitosa entre EC2 y RDS: Mediante la revisión de los registros (logs) del servidor de procesos PM2, se comprobó la correcta inicialización y establecimiento del pool de conexiones hacia la base de datos en la nube.

![](images/conec.png)

## 4.3. Validación de Operaciones CRUD y Persistencia
Para comprobar la ejecución de las operaciones sobre la API y la correcta aplicación de las reglas de negocio (como la validación de duplicados en los títulos), se realizaron peticiones HTTP seguras ([https://benjamin-pereira.rootcode.com.bo/movies](https://benjamin-pereira.rootcode.com.bo/movies)) utilizando Postman:

### Operación CREATE (POST) y Validación de Duplicados:
Se evaluó el comportamiento de la API al intentar registrar una película existente, obteniendo una respuesta controlada de error (400 Bad Request) con el mensaje "El título ya existe para otra película", lo cual confirma la correcta ejecución de los middlewares de validación. Posteriormente, al registrar un título nuevo (ej. The Dark Knight), la API procesó e insertó el registro de forma exitosa.
![](images/post.png)

### Operación READ (GET) - Consulta y Listado:
Se ejecutó una petición de lectura sobre el endpoint /movies y rutas individuales por ID, obteniendo la estructura JSON correspondiente con los datos almacenados en Amazon RDS.
![](images/get1.png)
![](images/get2.png)

### Operaciones UPDATE (PUT) y DELETE (Borrado):
Se validó la modificación de registros existentes mediante solicitudes HTTP PUT y la eliminación de los mismos mediante DELETE, comprobando de manera empírica que los cambios impactan directamente y de forma persistente en la base de datos de Amazon RDS.

![](images/put.png)
![](images/delete.png)


La implementación y validación individual demostraron que la arquitectura despliega una API robusta, tolerante a fallos y segura. La integración de restricciones estrictas en los grupos de seguridad de AWS, sumada al cifrado TLS de Cloudflare y la correcta estructuración del CRUD en Node.js, garantiza la integridad, confidencialidad y persistencia de la información en entornos de producción en la nube.
![](images/fin.png)