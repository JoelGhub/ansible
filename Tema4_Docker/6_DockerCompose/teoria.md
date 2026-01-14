# Orquestación básica con docker-compose

`docker-compose` es una herramienta que permite definir y gestionar aplicaciones multicontenedor usando un archivo YAML (`docker-compose.yml`). Facilita el despliegue, escalado y gestión de servicios relacionados.

## Sintaxis básica de docker-compose.yml
Un archivo típico define servicios, redes y volúmenes. Ejemplo:
```yaml
version: '3.8'
services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: ejemplo
    volumes:
      - db_data:/var/lib/mysql
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    depends_on:
      - db
volumes:
  db_data:
```

## Comandos principales
- `docker-compose up -d`: Levanta todos los servicios en segundo plano.
- `docker-compose down`: Detiene y elimina los servicios, redes y volúmenes creados.
- `docker-compose ps`: Lista los servicios activos.
- `docker-compose logs`: Muestra los logs de todos los servicios.
- `docker-compose exec <servicio> bash`: Accede a la terminal de un servicio.

## Buenas prácticas
- Usa variables de entorno para credenciales y configuraciones sensibles.
- Define volúmenes para persistencia de datos.
- Utiliza `depends_on` para gestionar dependencias entre servicios.

## Contexto
docker-compose es ideal para entornos de desarrollo, pruebas y despliegues sencillos. Para orquestación avanzada y en producción, se recomienda usar Kubernetes o Docker Swarm.
