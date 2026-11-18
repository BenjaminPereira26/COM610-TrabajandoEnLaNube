# Laboratorio 5.2: Configuración de un Pipeline de Despliegue Continuo con Docker

## Ejercicio 3.1: Creación del Dockerfile y prueba local

**Objetivo:** Contenerizar la aplicación Node.js utilizando un `Dockerfile` optimizado con múltiples etapas (multi-stage build), validar su funcionamiento localmente y preparar el código para el despliegue.

### 1. Ajuste del código base (Health Check)
Para garantizar que el pipeline de despliegue posterior pueda validar el estado de la aplicación (Health Check), se agregó el endpoint `/health` al archivo `app.js`. Este endpoint retorna un estado 200 y es vital para que el script de despliegue (Ejercicio 3.3) sepa que el contenedor está sano antes de exponerlo al tráfico público.

```javascript
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});
```

![](images/img1.png)

### 2. Creación del Dockerfile (Multi-stage)
Se creó un archivo Dockerfile en la raíz del proyecto. Se optó por un build de múltiples etapas (multi-stage):

* Etapa 1 (builder): Utiliza la imagen node:20-alpine para instalar únicamente las dependencias de producción (npm ci --only=production), evitando copiar herramientas de desarrollo (como Jest o ESLint) a la imagen final.

* Etapa 2 (runtime): Una imagen limpia de node:20-alpine que solo copia los módulos de Node y el código fuente necesario (app.js, server.js). Esto reduce drásticamente el peso de la imagen final y mejora la seguridad.

![](images/img2.png)

### 3. Configuración del .dockerignore
Para evitar que archivos innecesarios o pesados se copien al contexto de construcción (lo que haría el build más lento), se creó el archivo .dockerignore excluyendo carpetas como node_modules, .git, .github y coverage.

![](images/img3.png)

### 4. Prueba local y validación
Se construyó la imagen localmente y se ejecutó un contenedor para verificar que la aplicación funcionaba correctamente:

![](images/img4.png)

![](images/img5.png)

Se verificó que el contenedor respondía correctamente con el JSON { "status": "ok" }.

Finalmente, se detuvo y eliminó el contenedor de prueba, y se subieron los archivos (Dockerfile, .dockerignore y el ajuste en app.js) al repositorio mediante una rama de características y una Pull Request.

![](images/img6.png)

![](images/img7.png)

![](images/img8.png)

## Ejercicio 3.2: Configuración de Secretos en GitHub
Se configuraron los siguientes secretos en el repositorio (Settings > Secrets and variables > Actions) para que el workflow pudiera autenticarse de forma segura:
*   `DOCKER_USERNAME` y `DOCKER_PASSWORD` (Access Token).
*   `SSH_HOST` (IP pública de la EC2: 3.86.55.238).
*   `SSH_USER` (ubuntu).
*   `SSH_PRIVATE_KEY` (Contenido de la llave .pem).


![](images/img9.png)

![](images/img12.png)

![](images/img19.png)

## Ejercicio 3.3: Workflow de Build y Push (Implementación)

### 1. Configuración de la Instancia EC2
Para el entorno de despliegue, se creó una instancia en AWS EC2 con sistema operativo Ubuntu. Se configuró el grupo de seguridad para permitir tráfico SSH (puerto 22), HTTP (puerto 80) y HTTPS (puerto 443). Una vez lanzada la instancia, se accedió vía SSH para instalar Docker y habilitar el servicio.

![](images/img17.png)

![](images/img18.png)




### 2. Workflow de CD (`cd.yml`)
Se creó el archivo `.github/workflows/cd.yml` dividido en dos jobs secuenciales:
1.  **build-and-push:** Se autentica en Docker Hub y construye la imagen utilizando el `Dockerfile` multi-stage, etiquetándola con `latest` y el SHA del commit.
2.  **deploy:** Utiliza la acción `appleboy/ssh-action` para conectarse al servidor EC2. El script descarga la nueva imagen, levanta un contenedor temporal en el puerto 3001 y realiza un **Health Check** (`curl /health`). Si el chequeo es exitoso, detiene el contenedor anterior y expone el nuevo en el puerto 80. Si falla, cancela el despliegue manteniendo la versión anterior activa.

![](images/img10.png)

![](images/img11.png)

![](images/img13.png)


![](images/img15.png)

![](images/img16.png)

![](images/img20.png)

![](images/img21.png)

![](images/img22.png)


### 3. Verificación del Despliegue
Tras realizar el `push` y fusionar la Pull Request, el pipeline se ejecutó automáticamente. Ambos jobs finalizaron exitosamente.

![](images/img14.png)

Finalmente, se accedió a la IP pública de la instancia EC2 desde el navegador, confirmando que la API estaba respondiendo correctamente en producción.

![](images/img23.png)

### Ejercicio 3.4: Verificación del despliegue y Rollbacks

### 1. Actualización de versión y despliegue exitoso
Para comprobar el funcionamiento del Despliegue Continuo, se modificó el endpoint `/health` en el archivo `app.js` para que devolviera un mensaje de versión diferente:


![](images/img24.png)

![](images/img25.png)

![](images/img26.png)

![](images/img27.png)

![](images/img28.png)

Tras subir el cambio mediante una Pull Request y fusionarlo, el pipeline de CD se ejecutó automáticamente. Los jobs de build-and-push y deploy finalizaron en verde (✅), actualizando el contenedor en la instancia EC2. Al acceder a la IP pública, se confirmó que la nueva versión estaba en línea.

### 2. Simulación de fallo en el Health Check
Para probar la seguridad del pipeline, se introdujo un error intencional modificando el puerto interno del archivo server.js a 9999 en lugar de 3000. Esto haría que el contenedor no respondiera en el puerto esperado.

![](images/img29.png)

![](images/img30.png)

![](images/img31.png)

Al subir este cambio, el job de deploy falló (❌). El script de despliegue intentó levantar el nuevo contenedor en el puerto 3001, pero el comando curl /health falló. Al detectar esto, el script canceló el despliegue y no reemplazó el contenedor activo.

![](images/img32.png)

Validación de la resiliencia: A pesar de que el pipeline de despliegue falló, se comprobó en el navegador que la versión anterior (2.0.0) seguía funcionando perfectamente en el puerto 80. Esto demuestra que el Health Check evitó que los usuarios experimentaran una caída del servicio (Downtime).

![](images/img33.png)

### 3. Estrategia de Rollback (Reversión)
El laboratorio requiere entender cómo revertir a una versión anterior. Gracias a que el workflow etiqueta cada imagen Docker con el SHA del commit (${{ github.sha }}), es posible hacer un rollback manual en caso de descubrir un bug después del despliegue.

```javascript
# Reemplaza SHA_ANTERIOR por el SHA del commit anterior
docker pull TU_USUARIO/mi-app:SHA_ANTERIOR
docker stop mi-app
docker rm mi-app
docker run -d --name mi-app --restart unless-stopped -p 80:3000 TU_USUARIO/mi-app:SHA_ANTERIOR
```

### 4. Corrección y despliegue final
Finalmente, se corrigió el puerto en server.js de vuelta a 3000. Se subió el cambio, y el pipeline de CD volvió a ejecutarse exitosamente (✅), confirmando que la aplicación quedó desplegada y estable en la instancia EC2.

![](images/img34.png)

![](images/img35.png)

![](images/img36.png)

## 4. Práctica Individual - Laboratorio 5.2

### 4.1. Descripción del Pipeline
Para la práctica individual se extendió el pipeline de CD del proyecto CRUD (desarrollado en el Laboratorio 5.1). Se implementaron dos mejoras clave:
1.  **Separación de entornos:** Uso de GitHub Environments (`staging` y `production`) para gestionar el despliegue de forma controlada.
2.  **Seguridad:** Integración del escáner de vulnerabilidades **Trivy** para analizar la imagen Docker antes de desplegarla.

### 4.2. Configuración de GitHub Environments
Se crearon los entornos `staging` y `production` en la configuración del repositorio. El job de `deploy` fue modificado para apuntar al entorno `production`, lo que permite en el futuro agregar reglas de protección (como requerir aprobación manual antes de desplegar).


![](images/img37.png)

### 4.3. Escaneo de Vulnerabilidades con Trivy
Se agregó el paso `aquasecurity/trivy-action` al workflow. Este paso analiza la imagen Docker recién construida en busca de vulnerabilidades críticas o altas (CVEs) en el sistema operativo y en las librerías de Node.js.

![](images/img38.png)

![](images/img39.png)

![](images/img40.png)

![](images/img41.png)

![](images/img42.png)

![](images/img43.png)

![](images/img44.png)

### 4.4. Evidencias del Despliegue Final
El pipeline finalizó exitosamente, desplegando la imagen segura en la instancia EC2. La aplicación se encuentra operativa y accesible vía HTTP.

![](images/img45.png)