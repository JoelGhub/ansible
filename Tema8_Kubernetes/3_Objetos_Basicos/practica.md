# Práctica: Objetos básicos de Kubernetes

1. Crea un manifiesto YAML para un Pod que ejecute nginx.
2. Aplica el manifiesto y comprueba el estado del Pod:
   ```bash
   kubectl apply -f mi-pod.yaml
   kubectl get pods
   kubectl describe pod mi-pod
   ```
3. Crea un Deployment y un Service para exponer el Pod.
4. Reflexiona: ¿Qué ventajas tiene usar Deployments frente a Pods sueltos?
