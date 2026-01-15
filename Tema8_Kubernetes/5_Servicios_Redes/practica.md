# Práctica: Servicios y redes en Kubernetes

1. Crea un manifiesto YAML para un Service de tipo NodePort que exponga nginx en el puerto 30080.
2. Aplica el manifiesto y comprueba el acceso desde el navegador:
   ```bash
   kubectl apply -f mi-servicio.yaml
   kubectl get services
   ```
3. Crea una política de red que limite el acceso entre Pods.
4. Reflexiona: ¿Por qué es importante controlar el tráfico entre servicios?
