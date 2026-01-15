# Práctica: Despliegue de aplicaciones en Kubernetes

1. Crea un manifiesto YAML para un Deployment con 3 réplicas de nginx.
2. Aplica el manifiesto y comprueba el estado de los Pods:
   ```bash
   kubectl apply -f mi-deployment.yaml
   kubectl get deployments
   kubectl get pods
   ```
3. Actualiza la imagen del Deployment y observa el proceso de rolling update.
4. Escala el número de réplicas a 5 y comprueba el resultado.
5. Reflexiona: ¿Qué ventajas aporta el despliegue automatizado frente al manual?
