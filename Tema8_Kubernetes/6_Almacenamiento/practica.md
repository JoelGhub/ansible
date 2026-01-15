# Práctica: Almacenamiento en Kubernetes

1. Crea un manifiesto YAML para un PersistentVolumeClaim de 1Gi.
2. Aplica el manifiesto y comprueba el estado del PVC:
   ```bash
   kubectl apply -f mi-pvc.yaml
   kubectl get pvc
   ```
3. Asocia el PVC a un Pod y verifica la persistencia de datos.
4. Reflexiona: ¿Qué ventajas tiene el almacenamiento dinámico frente al estático?
