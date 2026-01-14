# Gestión de Imágenes y Contenedores

La gestión de imágenes y contenedores es fundamental para trabajar con Docker. A continuación se explican los conceptos y comandos más importantes:

## Imágenes
Las imágenes son plantillas inmutables que contienen el sistema de archivos y las dependencias necesarias para ejecutar una aplicación. Se pueden descargar de repositorios públicos (como Docker Hub) o crear de forma personalizada.

### Comandos básicos de imágenes
- `docker pull <imagen>`: Descargar una imagen desde un repositorio remoto.
- `docker images`: Listar todas las imágenes locales.
- `docker rmi <id|nombre>`: Eliminar una imagen local.
- `docker tag <imagen> <nuevo_nombre>`: Etiquetar una imagen para subirla a un repositorio.

## Contenedores
Un contenedor es una instancia en ejecución de una imagen. Es efímero, portable y puede ser gestionado fácilmente.

### Comandos básicos de contenedores
- `docker run <imagen>`: Crear y ejecutar un contenedor a partir de una imagen.
- `docker ps`: Listar contenedores en ejecución.
- `docker ps -a`: Listar todos los contenedores (activos y detenidos).
- `docker stop <id|nombre>`: Detener un contenedor en ejecución.
- `docker start <id|nombre>`: Iniciar un contenedor detenido.
- `docker rm <id|nombre>`: Eliminar un contenedor.
- `docker logs <id|nombre>`: Ver los logs de un contenedor.
- `docker exec -it <id|nombre> bash`: Acceder a la terminal de un contenedor en ejecución.

## Ejemplo de flujo de trabajo
1. Descargar una imagen oficial de nginx: `docker pull nginx`
2. Ejecutar un contenedor: `docker run --name web1 -d -p 8080:80 nginx`
3. Verificar que está en ejecución: `docker ps`
4. Detener y eliminar el contenedor: `docker stop web1 && docker rm web1`

## Buenas prácticas
- Elimina imágenes y contenedores que no uses para ahorrar espacio.
- Usa nombres descriptivos para tus contenedores.
- Consulta los logs para depurar problemas.
- Mantén tus imágenes actualizadas y seguras.
