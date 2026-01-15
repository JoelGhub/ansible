# Práctica: Orquestación avanzada en Kubernetes

1. Crea un manifiesto YAML para un HorizontalPodAutoscaler.
2. Aplica el manifiesto y comprueba el escalado automático:
   ```bash
   kubectl apply -f mi-hpa.yaml
   kubectl get hpa
   ```
3. Realiza una actualización rolling update en un Deployment y observa el proceso.
4. Programa un CronJob que imprima la fecha cada minuto.
5. Reflexiona: ¿Qué ventajas aporta la orquestación avanzada en entornos reales?
