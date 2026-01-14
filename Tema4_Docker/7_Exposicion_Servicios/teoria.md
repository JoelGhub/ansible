# Exposición de Servicios (puertos, redes)

Exponer servicios en Docker permite que las aplicaciones dentro de los contenedores sean accesibles desde el exterior o desde otros contenedores. Esto se logra mediante el mapeo de puertos y la configuración de redes.

## Mapeo de puertos
- `-p <puerto_host>:<puerto_contenedor>`: Asocia un puerto del host con uno del contenedor.
	- Ejemplo: `docker run -p 8080:80 nginx` expone el puerto 80 del contenedor en el 8080 del host.
- Puedes mapear varios puertos y usar diferentes rangos.

## Redes personalizadas
Permiten que varios contenedores se comuniquen entre sí de forma segura y aislada. Es posible definir redes bridge, overlay, etc.

## Ejemplo de exposición de servicios
1. Ejecutar un contenedor de nginx y exponer el puerto 8080:
	 ```bash
	 docker run -d --name web1 -p 8080:80 nginx
	 ```
2. Acceder a `http://localhost:8080` desde el navegador.
3. Ejecutar varios contenedores en la misma red y comunicar servicios internos.

## Contexto
La exposición de servicios es fundamental para aplicaciones web, APIs y microservicios. Permite el acceso externo y la integración entre componentes distribuidos.
