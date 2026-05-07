# Pipeline CI/CD

## Descripción del proyecto

Este proyecto implementa un flujo completo de Integración Continua y Despliegue Continuo (CI/CD) utilizando Jenkins, Docker, SonarQube y Trivy sobre el repositorio:

El objetivo del ejercicio es automatizar:

* construcción de la aplicación,
* pruebas,
* análisis de calidad,
* análisis de seguridad,
* construcción de imágenes Docker,
* y despliegue automático local.


## Configuración del entorno

### 1. Jenkins en Docker

Se utilizó Jenkins ejecutándose en un contenedor Docker.

Comando utilizado:

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```

### 2. Plugins instalados en Jenkins

Se instalaron los siguientes plugins:

* Git
* Pipeline
* Docker Pipeline
* SonarQube Scanner
* Workspace Cleanup

### 3. SonarQube

Se ejecutó SonarQube localmente usando Docker.

Comando utilizado:

```bash
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  sonarqube:lts-community
```

Acceso:

```text
http://localhost:9000
```

### 4. Trivy

Trivy se utilizó mediante contenedor Docker para escanear vulnerabilidades en la imagen generada.

Imagen utilizada:

```text
aquasec/trivy:latest
```

## Arquitectura del pipeline

El Jenkinsfile desarrollado para este proyecto define un pipeline declarativo compuesto por varias etapas automatizadas que permiten ejecutar el flujo completo de integración continua y despliegue continuo.

1. La primera etapa corresponde a **Checkout**, donde Jenkins obtiene automáticamente el código fuente del proyecto desde el repositorio de GitHub configurado.
2. Después se ejecuta la etapa **Build & Test**, encargada de compilar la aplicación utilizando Maven dentro de un contenedor Docker basado en la imagen `maven:3.9.9-eclipse-temurin-17`. Durante esta fase se genera el paquete de la aplicación y se validan los procesos básicos de construcción.
3. La siguiente etapa es **Static Analysis (SonarQube)**. En esta fase Jenkins ejecuta un análisis estático del código fuente utilizando SonarQube con el objetivo de identificar problemas de calidad, vulnerabilidades, código duplicado y posibles malas prácticas de desarrollo.
4. Posteriormente se ejecuta la etapa **Quality Gate**, la cual valida automáticamente los resultados entregados por SonarQube. Si el proyecto no cumple las reglas mínimas de calidad configuradas, el pipeline se detiene automáticamente y evita continuar con el despliegue.
5. Luego se ejecuta la etapa **Docker Build**, donde Jenkins construye una imagen Docker de la aplicación utilizando el Dockerfile presente en el proyecto. La imagen generada se almacena con el nombre:

```text id="5c52jw"
cicd-demo:latest
```
6. Después se ejecuta la etapa **Container Security Scan (Trivy)**. En esta fase se utiliza Trivy para analizar la imagen Docker generada y detectar vulnerabilidades de seguridad conocidas. El pipeline está configurado para fallar automáticamente si Trivy encuentra vulnerabilidades de severidad crítica (CRITICAL).
7. Finalmente, se ejecuta la etapa **Deploy**, encargada de desplegar automáticamente la aplicación utilizando Docker. Antes de iniciar el nuevo contenedor, el pipeline elimina cualquier contenedor previo llamado `mi-app` para evitar conflictos. Luego se crea un nuevo contenedor utilizando la imagen construida previamente.

La aplicación queda desplegada localmente y accesible desde:

```text id="l4qv4h"
http://localhost:8081
```


## Automatización del pipeline

Jenkins fue configurado para detectar cambios automáticamente utilizando:

```text
Poll SCM
```

Configuración utilizada:

```text
* * * * *
```

Esto permite que Jenkins revise el repositorio cada minuto y ejecute automáticamente el pipeline después de un push.


## Manejo de errores

El pipeline incluye un bloque `post` para:

* limpiar el workspace,
* detectar fallos,
* mostrar mensajes de éxito o error.

```groovy
post {
    always {
        cleanWs()
    }

    failure {
        echo 'Pipeline falló'
    }

    success {
        echo 'Pipeline exitoso'
    }
}
```