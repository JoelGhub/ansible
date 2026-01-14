# Práctica: Arquitectura de Docker

## 1. Instalación de Docker
Sigue la documentación oficial para instalar Docker en tu sistema operativo:
- [Docker para Windows](https://docs.docker.com/desktop/install/windows-install/)
- [Docker para Mac](https://docs.docker.com/desktop/install/mac-install/)
- [Docker para Linux](https://docs.docker.com/engine/install/)

Verifica la instalación ejecutando:
```bash
docker --version
docker info
```

## 2. Explora los componentes principales
1. Comprueba que el servicio Docker está activo:
	```bash
	sudo systemctl status docker
	# o en Mac/Windows: comprobar que Docker Desktop está en ejecución
	```
2. Usa el comando `docker info` para ver detalles sobre el daemon, almacenamiento, redes y plugins.
3. Ejecuta tu primer contenedor:
	```bash
	docker run hello-world
	```
	Explica qué ocurre en este proceso (descarga de imagen, creación y ejecución del contenedor, salida por pantalla).

## 3. Ejercicios de exploración
- Lista las imágenes disponibles en tu sistema: `docker images`
- Lista los contenedores activos y detenidos: `docker ps -a`
- Explora las redes creadas por defecto: `docker network ls`
- Crea y elimina un volumen: `docker volume create prueba && docker volume rm prueba`

## 4. Reflexión
- ¿Qué ventajas aporta Docker frente a la instalación tradicional de aplicaciones?
- ¿Qué diferencias observas entre un contenedor y una máquina virtual?
