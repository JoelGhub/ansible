# Definición de Redes en Docker

Docker permite crear redes virtuales para que los contenedores se comuniquen entre sí y con el exterior. Las redes facilitan la seguridad, el aislamiento y la escalabilidad de las aplicaciones.

## Tipos de redes en Docker
- **bridge**: Red por defecto para contenedores en un solo host. Permite la comunicación entre contenedores y con el host.
- **host**: El contenedor comparte la pila de red del host (sin aislamiento de red).
- **none**: El contenedor no tiene acceso a ninguna red.
- **overlay**: Permite la comunicación entre contenedores en diferentes hosts (usado en Docker Swarm).
- **macvlan**: Asigna una dirección MAC a cada contenedor, útil para integrarse con redes físicas.

## Comandos útiles
- `docker network ls`: Listar todas las redes disponibles.
- `docker network inspect <nombre>`: Ver detalles de una red.
- `docker network create <nombre>`: Crear una red personalizada.
- `docker network rm <nombre>`: Eliminar una red.

## Ejemplo de uso
1. Crear una red personalizada:
	```bash
	docker network create red_practica
	```
2. Ejecutar dos contenedores en la misma red:
	```bash
	docker run -d --name cont1 --network red_practica alpine sleep 1000
	docker run -d --name cont2 --network red_practica alpine sleep 1000
	```
3. Comprobar la conectividad entre ellos:
	```bash
	docker exec cont1 ping cont2
	```

## Contexto
El uso de redes personalizadas es esencial para aplicaciones multicontenedor, microservicios y entornos de producción, ya que permite controlar el tráfico y la seguridad entre servicios.
