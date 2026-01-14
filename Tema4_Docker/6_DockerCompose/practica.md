# Práctica: docker-compose

## 1. Crear un archivo docker-compose.yml
1. Crea un directorio de trabajo y entra en él:
	```bash
	mkdir practica_compose && cd practica_compose
	```
2. Crea un archivo `docker-compose.yml` con el siguiente contenido:
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

## 2. Levantar y gestionar los servicios
1. Levanta los servicios en segundo plano:
	```bash
	docker-compose up -d
	```
2. Comprueba que ambos servicios están activos:
	```bash
	docker-compose ps
	```
3. Accede a la web en `http://localhost:8080` y verifica que responde.
4. Consulta los logs de ambos servicios:
	```bash
	docker-compose logs
	```
5. Accede a la terminal del contenedor web:
	```bash
	docker-compose exec web bash
	```
6. Detén y elimina los servicios, redes y volúmenes:
	```bash
	docker-compose down -v
	```

## 3. Ejercicios adicionales
- Añade un servicio adicional (por ejemplo, phpmyadmin) y configura su acceso.
- Modifica el archivo para exponer otro puerto o usar una red personalizada.

## 4. Reflexión
- ¿Qué ventajas aporta docker-compose frente a ejecutar contenedores manualmente?
- ¿Qué limitaciones tiene para entornos de producción?
