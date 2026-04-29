# Gestión de Submódulos Git
Este proyecto fue desarrollado utilizando submódulos de Git, por lo que es importante tener en cuenta los siguientes aspectos para trabajar correctamente con el repositorio.

## Clonar el Repositorio con Submódulos
Cuando clones el repositorio por primera vez, debes inicializar y actualizar los submódulos:

```
git clone <repository_url>
cd Human-Talent-Managment-App-Launcher
git submodule update --init --recursive
```

## Actualizar Referencias de Submódulos
Para obtener los últimos cambios de todos los submódulos:

git submodule update --remote

## Importante: Orden de Commits
Si trabajas en un repositorio que contiene submódulos:

1. Primero: Hacer push en el submódulo

```
cd <microservicio>
git add .
git commit -m "Cambios en el microservicio"
git push
```

2. Después: Hacer push en el repositorio principal

```
cd ..
git add .
git commit -m "Actualizar referencia del submódulo"
git push
```

**Nota**: Si se hace en orden inverso, se perderán las referencias de los submódulos y será necesario resolver conflictos.

# Instalación y Configuración
## Requisitos Previos
Docker y Docker Compose
Node.js 20+ (para desarrollo local)
Git

## Variables de entorno
1. Crea un archivo .env en la raíz del proyecto basado en .env.template
2. Configura las variables necesarias para cada microservicio
3. Asegúrate de tener las credenciales de:
  - Supabase
  - Cloudinary
  - Resend

