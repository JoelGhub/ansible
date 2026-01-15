# Orquestación avanzada en Kubernetes

Kubernetes permite gestionar aplicaciones complejas mediante técnicas avanzadas de orquestación y automatización.

## Funcionalidades avanzadas
- **Autoscaling:** Escalado automático de Pods y nodos según la carga.
- **Rolling updates:** Actualizaciones sin interrupciones.
- **Rollback:** Reversión automática ante fallos.
- **Jobs y CronJobs:** Ejecución de tareas puntuales o programadas.
- **Affinity y Tolerations:** Control avanzado de ubicación de Pods.
- **Helm:** Gestor de paquetes para Kubernetes.

## Ejemplo de autoscaling (YAML)
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: mi-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: mi-app
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

## Contexto
La orquestación avanzada permite optimizar recursos, mejorar la disponibilidad y automatizar la gestión de aplicaciones en producción.
