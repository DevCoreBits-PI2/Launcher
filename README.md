# Talent Core Launcher

Repositorio de orquestación del backend de **Talent Core**, una plataforma de gestión de talento humano. El sistema permite administrar usuarios, estructura organizacional, contratos, trayectoria laboral y evaluaciones de desempeño mediante una arquitectura de microservicios.

Este repositorio no contiene un frontend. Su función es agrupar los servicios como submódulos Git y facilitar su ejecución local con Docker Compose y su despliegue con imágenes de producción o Kubernetes.

## Funcionalidades principales

- Autenticación, invitaciones y gestión de administradores y empleados.
- Perfiles de empleados, imágenes y códigos QR temporales.
- Administración de áreas, cargos y jerarquía organizacional.
- Creación, renovación y seguimiento de contratos laborales.
- Registro de trayectoria y cambios laborales.
- Evaluaciones de desempeño y reportes por empleado o área.
- Notificaciones por correo para vencimientos de contratos.
- Documentación interactiva de la API con Swagger.

## Arquitectura

```text
Cliente HTTP
    |
    v
API Gateway :3000
    |
    v
NATS :4222
    |-- administrative-data-ms --> PostgreSQL, Cloudinary y Resend
    |-- users-ms               --> PostgreSQL, Supabase Auth y Cloudinary
    `-- trajectory-ms          --> PostgreSQL
```

El gateway es el único servicio que expone una API HTTP. La comunicación interna utiliza el patrón request/reply de NATS.

| Componente | Responsabilidad |
| --- | --- |
| [`gateway`](https://github.com/DevCoreBits-PI2/Gateway) | API REST, validación, autorización y documentación Swagger. |
| [`administrative-data-ms`](https://github.com/DevCoreBits-PI2/Administrative-Data-Microservice) | Áreas, cargos, contratos, archivos y alertas de vencimiento. |
| [`users-ms`](https://github.com/DevCoreBits-PI2/Users-Microservice) | Autenticación, administradores, empleados, perfiles y QR. |
| [`trajectory-ms`](https://github.com/DevCoreBits-PI2/Trajectory-Microservice) | Trayectoria laboral, evaluaciones y reportes de desempeño. |
| `nats-server` | Broker de mensajería entre el gateway y los microservicios. |

## Tecnologías

- Imágenes de Node.js 22 y TypeScript.
- NestJS 11.
- NATS.
- Prisma y PostgreSQL/Supabase.
- Supabase Auth.
- Cloudinary para archivos e imágenes.
- Resend para correo transaccional.
- Docker, Docker Compose, Helm y Kubernetes/GKE.
- Jest, ESLint y Prettier.

## Requisitos

- Git con soporte para submódulos.
- Docker y Docker Compose.
- Para desarrollo sin Docker: Node.js `20.19+`, `22.12+` o `24+` y npm.
- Bases de datos PostgreSQL/Supabase existentes para los tres dominios.
- Credenciales de Supabase, Cloudinary y Resend.

> El repositorio no incluye contenedores de base de datos ni migraciones. Las bases y sus esquemas deben existir antes de iniciar los servicios.

## Instalación

Clona el repositorio junto con sus submódulos:

```bash
git clone --recurse-submodules https://github.com/DevCoreBits-PI2/Launcher.git
cd Launcher
```

Si ya lo clonaste sin la opción anterior, inicializa los submódulos con:

```bash
git submodule update --init --recursive
```

Crea el archivo de configuración local:

```bash
cp .env.template .env
```

Completa los valores de `.env` antes de levantar la aplicación. Sustituye también los comentarios `/* ... */` de la plantilla, ya que no forman parte de la sintaxis dotenv. No confirmes este archivo en Git.

## Variables de entorno

| Variable | Descripción |
| --- | --- |
| `GATEWAY_PORT` | Puerto HTTP publicado por el gateway. |
| `DIRECT_URL_ADMINISTRATIVE_DATA_DB` | Conexión PostgreSQL del servicio administrativo. |
| `DIRECT_URL_USERS_DB` | Conexión PostgreSQL del servicio de usuarios. |
| `DIRECT_URL_TRAJECTORY_DB` | Conexión PostgreSQL del servicio de trayectoria. |
| `SUPABASE_URL` | URL del proyecto de Supabase. |
| `DATABASE_KEY` | Clave pública de Supabase. |
| `DATABASE_ADMIN_KEY` | Clave administrativa de Supabase; solo debe usarse en el servidor. |
| `CLOUDINARY_NAME` | Nombre del cloud de Cloudinary. |
| `CLOUDINARY_API_KEY` | API key de Cloudinary. |
| `CLOUDINARY_API_SECRET` | API secret de Cloudinary. |
| `RESEND_API_KEY` | API key de Resend. |
| `EMAIL_FROM` | Nombre del remitente de las notificaciones. |
| `EMAIL_FROM_ADDRESS` | Dirección de correo del remitente. |
| `REDIRECT_URL` | URL del frontend para completar el registro o inicio de sesión. |

## Ejecución local

Inicia todos los servicios en modo desarrollo:

```bash
docker compose up --build
```

Para ejecutarlos en segundo plano:

```bash
docker compose up --build -d
```

Para detenerlos:

```bash
docker compose down
```

Una vez iniciado el sistema:

- Estado del gateway: [http://localhost:3000/](http://localhost:3000/)
- Swagger: [http://localhost:3000/api/docs](http://localhost:3000/api/docs)
- Monitoreo de NATS: [http://localhost:8222/](http://localhost:8222/)

Si modificas `GATEWAY_PORT`, reemplaza `3000` en las URL anteriores por el puerto configurado.

## Desarrollo por servicio

Cada submódulo es un proyecto NestJS independiente. Para trabajar sin Docker, inicia primero NATS:

```bash
docker compose up nats-server
```

Luego, dentro del servicio que quieras desarrollar:

```bash
npm ci
npm run start:dev
```

Los servicios con Prisma ejecutan la generación del cliente antes de iniciar en modo desarrollo. Cada servicio necesita su propio archivo `.env`; consulta el `.env.template` del submódulo correspondiente.

## Comandos de calidad

Los comandos se ejecutan dentro de cada submódulo porque no existe un `package.json` en la raíz:

```bash
npm run build       # Compila el servicio
npm test            # Ejecuta pruebas unitarias
npm run test:e2e    # Ejecuta pruebas end-to-end
npm run test:cov    # Genera cobertura
npm run lint        # Ejecuta ESLint y aplica correcciones
npm run format      # Formatea con Prettier
```

> Las pruebas end-to-end incluidas son plantillas iniciales y todavía no representan los flujos actuales del sistema.

## Estructura del repositorio

```text
.
|-- gateway/                  # Submódulo: entrada HTTP
|-- administrative-data-ms/  # Submódulo: estructura y contratos
|-- users-ms/                # Submódulo: identidad y empleados
|-- trajectory-ms/           # Submódulo: trayectoria y desempeño
|-- k8s/talent-core/         # Chart de Helm y manifiestos para GKE
|-- docker-compose.yml       # Entorno local con recarga automática
|-- docker-compose.prod.yml  # Construcción de imágenes de producción
|-- nats-server.conf         # Configuración local de NATS
`-- .env.template            # Plantilla de variables del launcher
```

## Producción y Kubernetes

Para construir y ejecutar localmente las imágenes de producción:

```bash
docker compose -f docker-compose.prod.yml up --build
```

El directorio [`k8s/talent-core`](k8s/talent-core) contiene el chart de Helm usado para GKE. Antes de instalarlo deben existir en el clúster los secretos referenciados por los deployments y la configuración `nats-config`. Consulta [`k8s/k8s.README.md`](k8s/k8s.README.md) para los comandos operativos disponibles.

## Trabajo con submódulos

Para traer las referencias registradas por el launcher:

```bash
git submodule update --init --recursive
```

Para consultar actualizaciones remotas de todos los submódulos:

```bash
git submodule update --remote
```

Cuando realices cambios en un microservicio:

1. Confirma y publica primero los cambios dentro del submódulo.
2. Regresa a la raíz del launcher.
3. Confirma la nueva referencia del submódulo en este repositorio.

```bash
cd users-ms
git add .
git commit -m "Descripción del cambio"
git push

cd ..
git add users-ms
git commit -m "Actualizar referencia de users-ms"
git push
```

No publiques primero la referencia del launcher: otros colaboradores no podrán obtener un commit del submódulo que todavía no existe en su remoto.

## Licencia

Proyecto privado. Los paquetes están marcados como `UNLICENSED` y este repositorio no incluye una licencia de distribución.
