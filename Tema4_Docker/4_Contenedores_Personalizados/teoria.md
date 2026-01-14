# Creación de Contenedores Personalizados

Crear contenedores personalizados permite adaptar el entorno a las necesidades de tu aplicación. Para ello, se utiliza un archivo llamado `Dockerfile` que define los pasos para construir la imagen.

## ¿Qué es un Dockerfile?
Es un archivo de texto con instrucciones que indican cómo construir una imagen. Cada instrucción genera una nueva capa en la imagen.

### Instrucciones comunes en un Dockerfile
- `FROM`: Imagen base (por ejemplo, `FROM ubuntu:22.04`)
- `RUN`: Ejecuta comandos en la imagen (instalar paquetes, copiar archivos, etc.)
- `COPY` o `ADD`: Copia archivos/directorios al contenedor
- `WORKDIR`: Define el directorio de trabajo
- `CMD` o `ENTRYPOINT`: Comando por defecto al iniciar el contenedor

## Ejemplo de Dockerfile sencillo
```
FROM ubuntu:22.04
RUN apt update && apt install -y apache2
COPY index.html /var/www/html/index.html
CMD ["apachectl", "-D", "FOREGROUND"]
```
Este Dockerfile crea una imagen de Ubuntu con Apache y una página personalizada.

## Buenas prácticas
- Usa imágenes base ligeras (por ejemplo, `alpine` si es posible)
- Minimiza el número de capas
- Elimina archivos temporales para reducir el tamaño
- Usa variables de entorno para configuraciones

## Flujo de trabajo
1. Crea el Dockerfile en el directorio de tu proyecto
2. Añade los archivos necesarios (por ejemplo, `index.html`)
3. Construye la imagen:
	```bash
	docker build -t mi_apache .
	```
4. Ejecuta el contenedor:
	```bash
	docker run -d -p 8081:80 mi_apache
	```

## Contexto
Personalizar imágenes es esencial para aplicaciones empresariales, microservicios y despliegues automatizados. Permite reproducibilidad y portabilidad en cualquier entorno compatible con Docker.
