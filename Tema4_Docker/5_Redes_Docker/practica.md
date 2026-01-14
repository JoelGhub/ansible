# Práctica: Redes en Docker

## 1. Crear y explorar redes
1. Lista las redes existentes:
	```bash
	docker network ls
	```
2. Crea una red personalizada llamada `red_practica`:
	```bash
	docker network create red_practica
	```
3. Inspecciona la red creada:
	```bash
	docker network inspect red_practica
	```

## 2. Conectar contenedores a una red
1. Ejecuta dos contenedores Alpine en la misma red:
	```bash
	docker run -d --name cont1 --network red_practica alpine sleep 1000
	docker run -d --name cont2 --network red_practica alpine sleep 1000
	```
2. Desde `cont1`, haz ping a `cont2`:
	```bash
	docker exec cont1 ping -c 3 cont2
	```
3. Prueba a conectar un contenedor a varias redes y comprueba su conectividad.

## 3. Ejercicios adicionales
- Crea una red de tipo `bridge` y otra de tipo `macvlan` (si tu sistema lo permite).
- Elimina la red personalizada y observa qué ocurre con los contenedores conectados.

## 4. Reflexión
- ¿Por qué es importante aislar servicios en diferentes redes?
- ¿Qué ventajas aporta Docker frente a la configuración de red tradicional?
