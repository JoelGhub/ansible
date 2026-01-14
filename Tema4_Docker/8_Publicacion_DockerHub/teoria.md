# Publicación en repositorios como Docker Hub

Publicar imágenes en repositorios como Docker Hub permite compartir y distribuir tus aplicaciones de forma sencilla y global. Docker Hub es el registro público más popular, pero existen otros como GitHub Container Registry o repositorios privados.

## Pasos para publicar una imagen
1. Crea una cuenta gratuita en [Docker Hub](https://hub.docker.com/).
2. Inicia sesión desde la terminal:
	```bash
	docker login
	```
3. Etiqueta tu imagen siguiendo el formato `usuario/nombre_imagen:tag`:
	```bash
	docker tag mi_imagen usuario/mi_imagen:latest
	```
4. Sube la imagen al repositorio:
	```bash
	docker push usuario/mi_imagen:latest
	```
5. Comprueba en la web de Docker Hub que la imagen está disponible.

## Buenas prácticas
- Usa tags descriptivos para versiones (`latest`, `v1.0`, etc.)
- Añade una descripción y documentación a tu repositorio en Docker Hub.
- No subas imágenes con credenciales o datos sensibles.
- Usa repositorios privados para imágenes internas.

## Contexto
Publicar imágenes facilita la colaboración, el despliegue automatizado y la integración continua. Es fundamental para equipos DevOps y despliegues en la nube.
