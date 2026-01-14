# Práctica: Contenedores Personalizados

## 1. Crear un Dockerfile personalizado
1. Crea un directorio de trabajo y entra en él:
	```bash
	mkdir practica_apache && cd practica_apache
	```
2. Crea un archivo `index.html` con el contenido que prefieras.
3. Crea un archivo `Dockerfile` con el siguiente contenido:
	```Dockerfile
	FROM ubuntu:22.04
	RUN apt update && apt install -y apache2
	COPY index.html /var/www/html/index.html
	CMD ["apachectl", "-D", "FOREGROUND"]
	```

## 2. Construir y ejecutar la imagen
1. Construye la imagen:
	```bash
	docker build -t mi_apache .
	```
2. Ejecuta el contenedor exponiendo el puerto 8081:
	```bash
	docker run -d --name apache1 -p 8081:80 mi_apache
	```
3. Accede a `http://localhost:8081` y verifica que ves tu página personalizada.

## 3. Ejercicios adicionales
- Modifica el Dockerfile para instalar otro servidor web (por ejemplo, Nginx) y repite el proceso.
- Añade una variable de entorno al Dockerfile y muéstrala en la página web.
- Usa un archivo de configuración personalizado para Apache.

## 4. Reflexión
- ¿Qué ventajas tiene crear imágenes personalizadas frente a usar imágenes oficiales?
- ¿Qué problemas podrías encontrar al construir imágenes muy grandes?
