# Objetos básicos de Kubernetes

Kubernetes gestiona aplicaciones mediante objetos declarativos que definen el estado deseado del sistema.

## Principales objetos
- **Pod:** Unidad mínima que agrupa uno o varios contenedores.
- **Deployment:** Gestiona el despliegue y la actualización de Pods.
- **Service:** Expone Pods y gestiona el acceso a ellos.
- **ConfigMap:** Almacena configuraciones en formato clave-valor.
- **Secret:** Almacena información sensible (contraseñas, tokens).
- **Namespace:** Permite aislar recursos dentro del cluster.

## Ejemplo de manifiesto Pod (YAML)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

## Contexto
El uso de objetos permite automatizar y versionar la infraestructura como código.
