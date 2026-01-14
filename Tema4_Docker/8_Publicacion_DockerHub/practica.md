# Práctica: Publicación en Docker Hub

## 1. Publicar una imagen personalizada
1. Crea una cuenta en Docker Hub si no la tienes: https://hub.docker.com/
2. Inicia sesión desde la terminal:
	```bash
	docker login
	```
3. Etiqueta tu imagen personalizada (por ejemplo, `mi_apache`):
	```bash
	docker tag mi_apache usuario/mi_apache:latest
	```
4. Sube la imagen a tu repositorio:
	```bash
	docker push usuario/mi_apache:latest
	```
5. Comprueba en la web de Docker Hub que la imagen está disponible.

## 2. Ejercicios adicionales
- Publica una nueva versión de la imagen con cambios y usa un tag diferente.
- Elimina una imagen de tu repositorio desde la web de Docker Hub.
- Prueba a descargar tu imagen desde otro equipo o entorno.

## 3. Reflexión
- ¿Qué ventajas tiene usar un registro público frente a uno privado?
- ¿Qué riesgos existen al publicar imágenes en repositorios públicos?
