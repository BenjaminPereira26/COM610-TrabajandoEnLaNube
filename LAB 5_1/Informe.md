# Laboratorio 5.1: Creación de un Workflow de GitHub Actions para CI

## Ejercicio 3.1: Preparación del repositorio y la aplicación

**Objetivo:** Inicializar un proyecto de Node.js, crear una aplicación básica con Express, configurar pruebas unitarias con Jest y Supertest, y realizar el primer push al repositorio de GitHub.

### 1. Inicialización del proyecto y dependencias
Se clonó el repositorio y se inicializó el proyecto Node.js. Luego, se instalaron las dependencias necesarias (Express para el servidor, y Jest junto con Supertest para las pruebas).

![](images/img1.png)

![](images/img2.png)

![](images/img3.png)

![](images/img4.png)

```bash
npm init -y
npm install express
npm install --save-dev jest supertest
```
### 2. Creación de los archivos de la aplicación
Se crearon los archivos en la raíz del proyecto:

* app.js: Configuración de la aplicación Express y definición de las rutas /health y /users.

* server.js: Punto de entrada para levantar el servidor en el puerto 3000.

* app.test.js: Pruebas unitarias para validar el correcto funcionamiento de los endpoints.


![](images/img5.png)

![](images/img6.png)

![](images/img7.png)

### 3. Configuración de scripts en package.json

Se actualizó el archivo package.json para definir los comandos de inicio y pruebas, además de ajustar el punto de entrada (main).

![](images/img8.png)


```bash
"scripts": {
  "start": "node server.js",
  "test": "jest"
}
```

### 4. Ejecución de pruebas locales
Se ejecutaron las pruebas para validar que la aplicación funciona correctamente antes de subirla.

![](images/img9.png)

### 5. Control de versiones (.gitignore y primer push)
Se creó el archivo .gitignore para evitar subir la carpeta node_modules y el archivo .env al repositorio. Finalmente, se realizó el primer commit y se subieron los cambios a GitHub.

![](images/img10.png)

![](images/img11.png)

![](images/img12.png)

## Ejercicio 3.2: Creación del workflow de CI

**Objetivo:** Configurar un pipeline de Integración Continua (CI) con GitHub Actions para automatizar la ejecución de pruebas en cada push a la rama `main`.

### 1. Creación del archivo de workflow
Se creó la estructura de carpetas `.github/workflows/` y el archivo `ci.yml` en la raíz del repositorio. En este archivo se definió un flujo de trabajo básico que se dispara ante eventos de `push` y `pull_request` hacia la rama `main`.

![](images/img13.png)

![](images/img14.png)

### 2. Ejecución y verificación
Se realizó el commit y push de los cambios a GitHub. En la pestaña Actions del repositorio se verificó que el workflow se ejecutó exitosamente, mostrando un check verde (✅) en el historial de ejecuciones.

![](images/img15.png)

![](images/img16.png)

Posteriormente, se ingresó al detalle del job build-and-test. Se comprobó que cada uno de los pasos configurados (Checkout, Configurar Node, Instalar dependencias y Ejecutar pruebas) finalizó correctamente sin errores, validando así el correcto funcionamiento de la Integración Continua en el repositorio.

![](images/img17.png)

## Ejercicio 3.3: Extensión del pipeline con calidad de código

### 1. Configuración de ESLint
Se instaló ESLint como dependencia de desarrollo y se inicializó su configuración utilizando el asistente interactivo (`npx eslint --init`). 

![](images/img18.png)

Además, se agregó el script de linting en el archivo package.json:

![](images/img19.png)

### 2. Actualización del workflow de CI
Se modificó el archivo .github/workflows/ci.yml para incluir un nuevo paso llamado "Ejecutar linting", el cual se ejecuta inmediatamente después de instalar las dependencias y antes de correr las pruebas unitarias.

![](images/img20.png)

Se realizó el commit y push correspondiente. En la pestaña Actions, se verificó que el pipeline se ejecutó correctamente con el nuevo paso incluido (✅).

![](images/img21.png)

![](images/img22.png)

![](images/img23.png)

### 3. Prueba de fallo intencional (Validación del pipeline)
Para comprobar que el pipeline realmente detiene la ejecución si la calidad del código falla, se introdujo un error intencional en el archivo app.js, declarando una variable que no se utiliza en ninguna parte del código:

![](images/img24.png)

![](images/img25.png)

Al subir este cambio a GitHub, el workflow falló en el paso "Ejecutar linting". En el detalle del job se pudo observar el error no-unused-vars (variable declarada pero nunca utilizada), confirmando que la Integración Continua bloquea el flujo ante malas prácticas de código.

![](images/img26.png)

![](images/img27.png)

### 4. Corrección del error
Finalmente, se eliminó la variable no utilizada del archivo app.js, y se realizó un nuevo commit y push. Se verificó en la pestaña Actions que el pipeline volvió a ejecutarse exitosamente (✅), demostrando el ciclo completo de retroalimentación que ofrece la Integración Continua.

![](images/img28.png)

![](images/img29.png)

## Ejercicio 3.4: Uso de matrices de ejecución

### 1. Modificación del workflow para la matriz
Se actualizó el archivo `.github/workflows/ci.yml` para reemplazar la versión fija de Node.js por una estrategia de matriz (`strategy: matrix`). Esto permite que el job `build-and-test` se ejecute múltiples veces, una por cada versión definida.

Además, se agregó una variable de entorno para forzar el uso de Node 24 en las acciones internas de GitHub (`FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true`), tal como sugiere el laboratorio.

![](images/img30.png)

### 2. Ejecución y verificación de jobs paralelos
Se realizó el commit y push de los cambios a GitHub. En la pestaña Actions, se comprobó que el workflow generó dos jobs independientes y en paralelo, identificados por la versión de Node.js utilizada. Ambos jobs completaron todos los pasos (incluyendo linting y pruebas) de manera exitosa.

![](images/img31.png)

![](images/img32.png)

## 4. Práctica Individual

### 4.1. Descripción del Proyecto
Para esta práctica se desarrolló una **API CRUD (Create, Read, Update, Delete)** para la gestión de tareas (To-Do list) utilizando **Node.js** y el framework **Express**. La aplicación permite crear, listar, actualizar y eliminar tareas a través de endpoints HTTP, utilizando un arreglo en memoria como base de datos para facilitar las pruebas.

El proyecto incluye pruebas de integración desarrolladas con **Jest** y **Supertest**, las cuales validan el correcto funcionamiento de todos los endpoints. Además, se configuró ESLint para el análisis estático del código y se implementó un pipeline de Integración Continua con GitHub Actions, el cual genera un reporte de cobertura de código en cada ejecución.

### 4.2. Desarrollo de la Aplicación y Pruebas Locales
Se inicializó el proyecto Node.js, se instalaron las dependencias necesarias y se crearon los archivos base: `app.js` (lógica del CRUD), `server.js` (inicio del servidor), `app.test.js` (pruebas con Jest) y `eslint.config.mjs` (configuración del linter).

![](images/img33.png)

![](images/img34.png)

![](images/img35.png)

![](images/img36.png)

![](images/img37.png)

![](images/img38.png)

![](images/img39.png)

![](images/img40.png)

![](images/img41.png)

![](images/img42.png)

![](images/img43.png)

Para validar el funcionamiento de la API antes de subirla a GitHub, se ejecutaron las pruebas localmente con el comando `npm test`, verificando que las 5 pruebas de integración pasaran correctamente.


![](images/img44.png)

### 4.3. Pipeline de Integración Continua (GitHub Actions)
Se creó el archivo `.github/workflows/ci.yml` para automatizar el pipeline en cada `push` o `pull_request` hacia la rama `main`. El pipeline incluye los siguientes pasos:
1. **Checkout** del código.
2. **Setup** de Node.js v20.
3. **Instalación** de dependencias (`npm ci`).
4. **Linting** estático con ESLint.
5. **Ejecución de pruebas** con reporte de cobertura.
6. **Guardado del reporte** de cobertura como artefacto descargable.

Al subir el código, el workflow se ejecutó exitosamente. En la pestaña **Actions** se puede observar el historial de ejecuciones en verde (✅).

![](images/img45.png)

![](images/img46.png)

![](images/img47.png)

![](images/img48.png)

Al ingresar al detalle del job `build-and-test`, se comprobó que todos los pasos (incluyendo el linting y las pruebas con cobertura) finalizaron correctamente, y se generó el artefacto `reporte-cobertura`.

![](images/img49.png)

![](images/img50.png)

![](images/img51.png)

### 4.4. Protección de Rama y Pull Request
Para garantizar la calidad del código, se configuró una regla de protección en la rama `main` desde **Settings > Branches**. Se habilitó la opción **"Require status checks to pass before merging"** y se seleccionó el check `build-and-test` como obligatorio.

![](images/img52.png)

![](images/img53.png)

![](images/img54.png)

Posteriormente, se creó una rama de características (`feature/prueba-proteccion`), se realizó un cambio menor en el `README.md` y se abrió una Pull Request hacia `main`. Como se observa en la captura, GitHub bloqueó automáticamente el botón de merge mientras el CI estaba en progreso.

![](images/img55.png)

![](images/img56.png)

![](images/img57.png)

Una vez que el workflow terminó en verde (✅), el botón de **"Merge pull request"** se habilitó, permitiendo fusionar los cambios de forma segura tras validar que el código no rompía nada.

![](images/img58.png)

![](images/img59.png)

![](images/img60.png)

![](images/img61.png)

![](images/img62.png)

![](images/img63.png)

### 4.5. Conclusiones y Reflexión
La implementación de Integración Continua mediante GitHub Actions demostró ser una herramienta fundamental para garantizar la calidad del software. 

*   **Detección temprana de errores:** El pipeline atrapa errores de sintaxis, malas prácticas de código (linting) y fallos lógicos (pruebas unitarias) antes de que el código llegue a la rama principal.
*   **Cobertura de código:** La generación del reporte de cobertura permite identificar qué partes de la lógica de negocio no están siendo probadas, incentivando la mejora continua de las pruebas.
*   **Flujo de trabajo seguro:** La protección de rama junto con las Pull Requests garantiza que ningún código roto se integre al proyecto, actuando como una barrera de seguridad automatizada que simula un entorno de trabajo profesional en equipo.

Finalmente, se inició el servidor con `npm start` y se comprobó el funcionamiento del CRUD desde el navegador accediendo a `http://localhost:3000/tasks`, donde se pudo observar el JSON con las tareas creadas.

![](images/img64.png)

