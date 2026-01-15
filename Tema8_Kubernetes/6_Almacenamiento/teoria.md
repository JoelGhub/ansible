# Almacenamiento en Kubernetes

Kubernetes permite gestionar almacenamiento persistente para aplicaciones mediante volúmenes y objetos específicos.

## Tipos de almacenamiento
- **emptyDir:** Volumen temporal que se borra al eliminar el Pod.
- **hostPath:** Usa un directorio del nodo como volumen.
- **PersistentVolume (PV):** Recurso de almacenamiento gestionado por el cluster.
- **PersistentVolumeClaim (PVC):** Solicitud de almacenamiento por parte de una aplicación.
- **StorageClass:** Define tipos de almacenamiento dinámico (SSD, HDD, red, etc.).

## Ejemplo de PVC (YAML)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mi-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

## Contexto
El almacenamiento persistente es esencial para bases de datos, aplicaciones con estado y gestión de datos en la nube.
