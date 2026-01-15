# Arquitectura de Kubernetes

Kubernetes se basa en una arquitectura distribuida formada por varios componentes que trabajan juntos para gestionar el ciclo de vida de los contenedores.

## Componentes principales
- **API Server:** Punto de entrada para todas las peticiones al cluster.
- **Scheduler:** Asigna los contenedores a los nodos disponibles.
- **Controller Manager:** Supervisa el estado del cluster y realiza tareas automáticas.
- **etcd:** Base de datos distribuida que almacena la configuración y el estado del cluster.
- **Kubelet:** Agente que corre en cada nodo y gestiona los contenedores.
- **Kube-proxy:** Gestiona la red y el acceso a los servicios.

## Diagrama de arquitectura (ASCII)
```
+-------------------+
|   API Server      |
+-------------------+
| Scheduler         |
| ControllerManager |
| etcd              |
+-------------------+
        |
+-------+-------+
|       |       |
Kubelet Kubelet Kubelet
 Node   Node   Node
```

## Contexto
Esta arquitectura permite alta disponibilidad, escalabilidad y gestión automática de aplicaciones en contenedores.
