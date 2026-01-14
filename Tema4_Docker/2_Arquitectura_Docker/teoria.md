# Arquitectura de Docker

Docker es una plataforma de código abierto que automatiza el despliegue de aplicaciones dentro de contenedores. Su arquitectura está compuesta por varios elementos fundamentales:

## Componentes principales

- **Docker Engine:** Servicio principal que ejecuta y gestiona contenedores. Incluye:
	- **Docker Daemon (dockerd):** Proceso que gestiona imágenes, contenedores, redes y volúmenes.
	- **Docker CLI:** Herramienta de línea de comandos para interactuar con el daemon.
	- **API REST:** Permite la integración con otras herramientas y automatización.
- **Imágenes:** Plantillas inmutables que contienen el sistema de archivos y las dependencias necesarias para ejecutar una aplicación. Se almacenan en repositorios locales o remotos.
- **Contenedores:** Instancias en ejecución de imágenes. Son efímeros, portables y ligeros.
- **Redes:** Permiten la comunicación entre contenedores y con el exterior. Docker crea redes por defecto, pero se pueden definir redes personalizadas.
- **Volúmenes:** Sistemas de almacenamiento persistente para datos generados y usados por contenedores.
- **Docker Hub y otros registros:** Repositorios públicos o privados donde se almacenan y comparten imágenes.

## Diagrama de arquitectura (ASCII)

```
				 +-------------------+
				 |  Docker Engine    |
				 +-------------------+
				 |  CLI / API REST   |
				 +-------------------+
				 |  Docker Daemon    |
				 +-------------------+
				 | Imágenes          |
				 | Contenedores      |
				 | Redes             |
				 | Volúmenes         |
				 +-------------------+
```

## Ejemplo de flujo de trabajo
1. El usuario ejecuta un comando con Docker CLI (por ejemplo, `docker run nginx`).
2. El Docker Daemon busca la imagen localmente o la descarga de Docker Hub.
3. Se crea y ejecuta el contenedor.
4. El contenedor puede conectarse a redes, usar volúmenes y exponer puertos.

## Contexto actual
Docker se ha convertido en el estándar de facto para la contenerización de aplicaciones. Existen alternativas como Podman o containerd, pero Docker sigue siendo la opción más popular para desarrollo, pruebas y despliegue en producción.
