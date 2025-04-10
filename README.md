# Índice - Guía Esencial de Docker

- [Índice - Guía Esencial de Docker](#índice---guía-esencial-de-docker)
  - [1. Conceptos Básicos de Docker](#1-conceptos-básicos-de-docker)
  - [2. Docker CLI (Interfaz de Línea de Comandos)](#2-docker-cli-interfaz-de-línea-de-comandos)
  - [3. Dockerfile](#3-dockerfile)
  - [4. Docker Compose 🐳](#4-docker-compose-)
    - [📘 `docker-compose.yml`: Aprende a usar Docker Compose para definir y ejecutar aplicaciones multi-contenedor](#-docker-composeyml-aprende-a-usar-docker-compose-para-definir-y-ejecutar-aplicaciones-multi-contenedor)
    - [🧱 Ejemplo básico de `docker-compose.yml`](#-ejemplo-básico-de-docker-composeyml)
    - [Definir Servicios](#definir-servicios)
    - [💾 Volúmenes y Redes](#-volúmenes-y-redes)
    - [🌐 Redes: Comunicación entre contenedores](#-redes-comunicación-entre-contenedores)
    - [depends\_on](#depends_on)
    - [🎯 Ejemplo: Docker Compose con `depends_on`, `healthcheck` y `wait-for-it.sh`](#-ejemplo-docker-compose-con-depends_on-healthcheck-y-wait-for-itsh)
    - [🧠 Comandos Útiles de Docker Compose](#-comandos-útiles-de-docker-compose)
  - [5. Docker Volumes](#5-docker-volumes)
  - [6. Docker Networking](#6-docker-networking)
  - [7. Docker Registry](#7-docker-registry)
  - [8. Docker y Seguridad](#8-docker-y-seguridad)
  - [9. Gestión de Contenedores en Producción](#9-gestión-de-contenedores-en-producción)
  - [10. Docker en un Proyecto Real](#10-docker-en-un-proyecto-real)
  - [11. Buenas Prácticas](#11-buenas-prácticas)
  - [Resumen de los Pasos](#resumen-de-los-pasos)


## 1. Conceptos Básicos de Docker
- **Contenedores vs Máquinas Virtuales**: Entiende la diferencia fundamental entre contenedores y máquinas virtuales.
- **Docker Engine**: Aprende cómo funciona el motor de Docker, que permite crear, gestionar y ejecutar contenedores.
- **Docker Images**: Entiende cómo crear, usar y gestionar imágenes, que son las plantillas de los contenedores.
- **Docker Containers**: Aprende a crear, ejecutar, detener, eliminar y gestionar contenedores.

## 2. Docker CLI (Interfaz de Línea de Comandos)

- **docker run**: Cómo ejecutar un contenedor a partir de una imagen.
- **docker ps**: Ver los contenedores en ejecución.
- **docker stop / docker start / docker restart**: Gestionar el ciclo de vida de los contenedores.
- **docker exec**: Ejecutar comandos dentro de un contenedor en ejecución.
- **docker logs**: Ver los logs de un contenedor para depuración.
- **docker rm** y **docker rmi**: Eliminar contenedores e imágenes, respectivamente.
- **docker build**: Construir imágenes Docker a partir de un Dockerfile.

## 3. Dockerfile

- **Sintaxis del Dockerfile**: Aprende a escribir y entender un Dockerfile, que es el archivo de configuración para crear una imagen.
  - **FROM**: Definir la imagen base.
  - **RUN**: Ejecutar comandos durante la construcción de la imagen.
  - **COPY / ADD**: Copiar archivos/directorios al contenedor.
  - **WORKDIR**: Definir el directorio de trabajo dentro del contenedor.
  - **EXPOSE**: Exponer puertos del contenedor para su uso.
  - **CMD**: Definir el comando predeterminado que se ejecutará cuando inicie el contenedor.

## 4. Docker Compose 🐳

### 📘 `docker-compose.yml`: Aprende a usar Docker Compose para definir y ejecutar aplicaciones multi-contenedor

Docker Compose permite definir y manejar múltiples contenedores como una sola aplicación. Toda la configuración se realiza en un archivo llamado `docker-compose.yml`.

### 🧱 Ejemplo básico de `docker-compose.yml`

```yaml
version: "3.8"

services:
  app:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/app

  db:
    image: postgres:14
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydatabase
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### Definir Servicios

- Podés configurar cualquier servicio necesario: aplicaciones web, bases de datos, sistemas de caché, etc.

```yaml
services:
  web:
    build: ./web
    ports:
      - "8080:80"

  redis:
    image: redis:alpine
```

- Cada servicio puede tener su propia configuraciòn como:
  - Variables de entorno (**environment**)
  - Montaje de volúmenes (**volumes**)
  - Configuración de redes (**networks**)
  - Exposición de puertos (**ports**)

### 💾 Volúmenes y Redes

- 📦 **Volúmenes: Persistencia de datos**

    - Definición de volúmenes:

```yaml
volumes:
  pgdata:
```

- Asignación en servicios:

```yaml
services:
  db:
    volumes:
      - pgdata:/var/lib/postgresql/data
```

### 🌐 Redes: Comunicación entre contenedores

- Definición de redes:

```yaml
networks:
  backend:
```

- Asignación en servicios:

```yaml
services:
  app:
    networks:
      - backend

  db:
    networks:
      - backend
```

### depends_on

- La clave depends_on en un archivo docker-compose.yml se usa para indicar que un servicio depende de otro. Es decir, le dice a Docker Compose que debe iniciar primero el servicio del que depende.

```yaml
services:
  app:
    depends_on:
      - db
    networks:
      - backend

  db:
    networks:
      - backend
```

- 😬 ¡Pero ojo! depends_on no espera a que db esté "listo"
  Esto es MUY importante:
  depends_on solo asegura el orden de arranque, NO espera a que el servicio db esté completamente listo (por ejemplo, que PostgreSQL ya esté escuchando conexiones).

  - 🔧 Si necesitás esperar a que un servicio esté listo (por ejemplo, una DB que demore en inicializarse), lo ideal es:

  - Usar un script de espera como wait-for-it, dockerize o similares.O bien, implementar lógica de reintentos en tu app.

### 🎯 Ejemplo: Docker Compose con `depends_on`, `healthcheck` y `wait-for-it.sh`

- 🐳 docker-compose.yml

```yaml
services:
  app:
    build: ./app
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend
    command: ["./wait-for-it.sh", "db:5432", "--", "node", "index.js"]

  db:
    image: postgres:14
    restart: always
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  - backend:
```

### 🧠 Comandos Útiles de Docker Compose

| Comando                                    | Descripción                                                             |
| ------------------------------------------ | ----------------------------------------------------------------------- |
| `docker compose up`                        | Levanta todos los servicios definidos en el `docker-compose.yml`        |
| `docker compose up -d`                     | Levanta los servicios en segundo plano (modo "detached")                |
| `docker compose down`                      | Detiene y elimina los contenedores, redes y volúmenes anónimos          |
| `docker compose build`                     | Construye (o reconstruye) las imágenes definidas                        |
| `docker compose build <servicio>`          | Construye solo el servicio especificado                                 |
| `docker compose ps`                        | Muestra el estado de los servicios/contenedores                         |
| `docker compose logs`                      | Muestra los logs de salida de los servicios                             |
| `docker compose exec <servicio> <comando>` | Ejecuta comandos dentro de un contenedor (`docker exec` estilo Compose) |

## 5. Docker Volumes

- **Volúmenes Docker**: Aprende cómo los volúmenes permiten persistir datos en contenedores y compartir datos entre contenedores.
- **docker volume create**: Crear y gestionar volúmenes.
- **Montar volúmenes en contenedores**: Cómo montar volúmenes en contenedores para persistir datos (por ejemplo, bases de datos).

## 6. Docker Networking

- **Redes Docker**: Comprender cómo Docker maneja la comunicación entre contenedores.
- **Redes Bridge, Host y Overlay**: Conocer las redes predeterminadas y cómo crear redes personalizadas.
- **Exponer puertos**: Cómo exponer puertos de contenedores para acceder a servicios desde el exterior.

## 7. Docker Registry

- **Docker Hub**: Aprende a usar Docker Hub para almacenar y compartir imágenes.
- **Crear un Docker Registry privado**: Si necesitas tener un registro privado para imágenes.
- **docker pull** y **docker push**: Descargar imágenes desde un registro y subir imágenes a tu propio registro.

## 8. Docker y Seguridad

- **Permisos y acceso a contenedores**: Configurar seguridad para ejecutar contenedores con permisos limitados.
- **Imágenes seguras**: Aprender a usar imágenes oficiales y garantizar que las imágenes propias sean seguras.
- **Escanear imágenes**: Usar herramientas como **Clair** para escanear imágenes por vulnerabilidades.

## 9. Gestión de Contenedores en Producción

- **Docker Swarm (básico)**: Si necesitas orquestación a pequeña escala, Swarm es útil para gestionar un clúster de contenedores Docker.
- **Logs y monitoreo**: Aprende a integrar herramientas para monitorear y obtener logs de contenedores (como **ELK stack**, **Prometheus**, **Grafana**).
- **Ciclos de vida de los contenedores**: Cómo manejar el ciclo de vida de tus contenedores y actualizarlos de manera efectiva.

## 10. Docker en un Proyecto Real

- **Integración en proyectos de backend**: Cómo Docker ayuda a aislar tu aplicación backend, gestionar bases de datos y otros servicios como cache o colas de mensajes.
- **CI/CD con Docker**: Usar Docker en pipelines de integración continua y despliegue continuo.

## 11. Buenas Prácticas

- **Optimización de imágenes**: Reducir el tamaño de las imágenes Docker (usando imágenes base más ligeras, eliminando archivos temporales).
- **Manejo de secretos**: Cómo gestionar credenciales de manera segura dentro de contenedores.
- **Versionado de imágenes**: Usar etiquetas de versiones adecuadas para mantener un buen control sobre las versiones de imágenes.

## Resumen de los Pasos

1. **Conceptos básicos**: Comprende contenedores, imágenes y el Docker Engine.
2. **Docker CLI**: Familiarízate con los comandos para gestionar contenedores e imágenes.
3. **Dockerfile**: Aprende a escribir Dockerfiles para crear imágenes personalizadas.
4. **Docker Compose**: Aprende a gestionar aplicaciones con múltiples contenedores.
5. **Docker Volumes y Redes**: Administra datos persistentes y la comunicación entre contenedores.
6. **Docker Registry**: Almacena y comparte tus imágenes con Docker Hub o tu propio registro.
7. **Seguridad**: Aprende a asegurar tus contenedores y las imágenes que usas.
8. **Monitoreo**: Implementa herramientas de monitoreo y logs para contenedores en producción.
9. **Buenas prácticas**: Sigue las mejores prácticas para mantener tus imágenes seguras y optimizadas.

---
