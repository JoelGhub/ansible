# Servicios y redes en Kubernetes

Kubernetes gestiona la comunicación entre aplicaciones y el acceso externo mediante objetos Service y redes virtuales.

## Tipos de Service
- **ClusterIP:** Acceso interno dentro del cluster.
- **NodePort:** Expone el servicio en un puerto del nodo para acceso externo.
- **LoadBalancer:** Integra balanceadores de carga externos (en la nube).
- **ExternalName:** Redirige a un nombre DNS externo.

## Redes en Kubernetes
- Cada Pod tiene una IP única.
- Los servicios gestionan el descubrimiento y el acceso entre Pods.
- Se pueden definir políticas de red para controlar el tráfico.

## Ejemplo de Service (YAML)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-servicio
spec:
  type: NodePort
  selector:
    app: mi-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

## Contexto
La gestión de servicios y redes es clave para la escalabilidad y seguridad de aplicaciones distribuidas.
