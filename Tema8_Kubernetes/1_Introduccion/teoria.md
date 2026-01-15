# Introducción a Kubernetes

Kubernetes es una plataforma de orquestación de contenedores de código abierto que automatiza el despliegue, la gestión y el escalado de aplicaciones en contenedores. Fue desarrollada por Google y ahora es mantenida por la Cloud Native Computing Foundation (CNCF).

## ¿Por qué usar Kubernetes?
- Permite gestionar aplicaciones distribuidas y microservicios de forma eficiente.
- Automatiza tareas como el escalado, la recuperación ante fallos y la actualización de aplicaciones.
- Facilita la portabilidad entre diferentes entornos (local, nube, híbrido).

## Componentes principales
- **Cluster:** Conjunto de máquinas (nodos) que ejecutan Kubernetes.
- **Master:** Nodo que gestiona el estado del cluster y coordina las tareas.
- **Worker:** Nodo que ejecuta los contenedores de las aplicaciones.

## Diagrama básico (ASCII)
```
+-------------------+
|   Master Node     |
+-------------------+
        |
+-------+-------+
|       |       |
Worker Worker Worker
 Node   Node   Node
```

## Contexto actual
Kubernetes es el estándar de facto para la orquestación de contenedores en la industria, utilizado por empresas de todos los tamaños para gestionar aplicaciones modernas.
