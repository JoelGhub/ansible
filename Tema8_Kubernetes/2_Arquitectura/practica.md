# Práctica: Arquitectura de Kubernetes

1. Dibuja un diagrama propio de la arquitectura de Kubernetes y explica cada componente.
2. Instala Minikube o Kind y explora los logs del API Server y del Scheduler:
   ```bash
   kubectl logs -n kube-system <nombre_pod_api_server>
   kubectl logs -n kube-system <nombre_pod_scheduler>
   ```
3. Reflexiona: ¿Por qué es importante la base de datos etcd en el cluster?
