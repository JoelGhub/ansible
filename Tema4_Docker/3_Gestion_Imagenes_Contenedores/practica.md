# Práctica: Gestión de Imágenes y Contenedores

## 1. Primeros pasos con imágenes y contenedores
1. Descarga una imagen oficial de nginx:
	```bash
	docker pull nginx
	```
2. Ejecuta un contenedor llamado `web1` en segundo plano y expón el puerto 8080:
	```bash
	docker run --name web1 -d -p 8080:80 nginx
	```
3. Comprueba que el contenedor está en ejecución:
	```bash
	docker ps
	```
4. Accede a `http://localhost:8080` desde tu navegador y verifica que ves la página de bienvenida de nginx.

## 2. Gestión de contenedores
1. Detén el contenedor:
	```bash
	docker stop web1
	```
2. Inicia de nuevo el contenedor:
	```bash
	docker start web1
	```
3. Consulta los logs del contenedor:
	```bash
	docker logs web1
	```
4. Accede a la terminal del contenedor:
	```bash
	docker exec -it web1 bash
	```
	Escribe `exit` para salir.
5. Elimina el contenedor:
	```bash
	docker rm -f web1
	```

## 3. Gestión de imágenes
1. Lista todas las imágenes locales:
	```bash
	docker images
	```
2. Elimina una imagen que no necesites:
	```bash
	docker rmi <id_imagen>
	```

## 4. Ejercicio extra
- Crea y ejecuta un contenedor de Alpine Linux (`docker run -it alpine sh`), instala un paquete y elimina el contenedor.

## 5. Reflexión
- ¿Qué ventajas tiene usar contenedores frente a instalar aplicaciones directamente en tu sistema?
- ¿Qué ocurre si eliminas un contenedor pero no la imagen?
