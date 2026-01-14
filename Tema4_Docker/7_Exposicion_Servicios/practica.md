# Práctica: Exposición de Servicios

## 1. Exponer un servicio web
1. Ejecuta un contenedor de nginx exponiendo el puerto 8080:
	```bash
	docker run -d --name web1 -p 8080:80 nginx
	```
2. Accede a `http://localhost:8080` desde tu navegador o usa:
	```bash
	curl http://localhost:8080
	```

## 2. Exponer varios puertos
1. Ejecuta un contenedor de Apache exponiendo los puertos 8081 y 8443:
	```bash
	docker run -d --name apache1 -p 8081:80 -p 8443:443 httpd
	```
2. Comprueba el acceso a ambos puertos.

## 3. Comunicación entre contenedores
1. Crea una red personalizada:
	```bash
	docker network create red_web
	```
2. Ejecuta dos contenedores en la misma red y haz que uno consuma un servicio del otro (por ejemplo, usando curl entre contenedores).

## 4. Ejercicios adicionales
- Modifica la configuración de nginx para servir una página personalizada.
- Usa el comando `docker inspect` para ver los puertos expuestos y la IP interna del contenedor.

## 5. Reflexión
- ¿Por qué es importante exponer solo los puertos necesarios?
- ¿Qué riesgos de seguridad existen al exponer servicios?
