# Despliegue de aplicaciones en Kubernetes

Kubernetes permite desplegar aplicaciones de forma declarativa y automatizada, facilitando la gestión de versiones, escalado y actualizaciones.

## Proceso de despliegue
1. Crear manifiestos YAML para los objetos necesarios (Deployment, Service, ConfigMap, etc.).
2. Aplicar los manifiestos con `kubectl apply -f <archivo>.yaml`.
3. Kubernetes crea y gestiona los Pods según el estado deseado.
4. Se pueden escalar réplicas, actualizar imágenes y gestionar el ciclo de vida de la aplicación.

## Ejemplo de despliegue (YAML)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mi-app
  template:
    metadata:
      labels:
        app: mi-app
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

## Contexto
El despliegue automatizado permite alta disponibilidad, actualizaciones sin interrupciones y gestión eficiente de recursos.
