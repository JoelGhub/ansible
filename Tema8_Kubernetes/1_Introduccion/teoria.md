# Introducción a Kubernetes

Kubernetes (también conocido como K8s) es una plataforma de orquestación de contenedores de código abierto que automatiza el despliegue, la gestión y el escalado de aplicaciones en contenedores. Fue desarrollada originalmente por Google basándose en su experiencia con Borg y Omega (sistemas internos de Google para gestionar contenedores), y fue liberado como proyecto open source en 2014. Actualmente es mantenido por la Cloud Native Computing Foundation (CNCF).

## Historia y evolución
- **2003-2004:** Google desarrolla Borg, su sistema interno de gestión de contenedores
- **2014:** Google libera Kubernetes como proyecto open source
- **2015:** Kubernetes v1.0 y creación de la CNCF
- **2017-2018:** Adopción masiva por la industria, se convierte en estándar
- **2020-presente:** Madurez del ecosistema, integración con clouds y herramientas

## ¿Qué significa el nombre?
Kubernetes proviene del griego κυβερνήτης (kubernḗtēs), que significa "timonel" o "piloto". El número 8 en K8s representa las 8 letras entre la 'K' y la 's'.

## ¿Por qué usar Kubernetes?

### Ventajas principales
- **Orquestación automática:** Gestiona el ciclo de vida completo de los contenedores
- **Escalado automático:** Aumenta o reduce réplicas según la demanda
- **Auto-reparación:** Reinicia contenedores fallidos, reemplaza y reprograma contenedores
- **Gestión de configuración:** Almacena y gestiona información sensible (secrets, configmaps)
- **Balanceo de carga:** Distribuye el tráfico entre múltiples instancias
- **Despliegues sin tiempo de inactividad:** Rolling updates y rollbacks automáticos
- **Portabilidad:** Funciona en local, on-premise, cloud pública o híbrida
- **Extensibilidad:** API extensible y ecosistema rico de plugins

### Casos de uso reales
- **Microservicios:** Gestionar cientos de servicios independientes
- **Aplicaciones web escalables:** E-commerce, streaming, redes sociales
- **Machine Learning:** Entrenar y desplegar modelos a escala
- **Big Data:** Procesamiento distribuido de grandes volúmenes de datos
- **CI/CD:** Pipelines de integración y despliegue continuo
- **Multi-tenant:** Aislar aplicaciones de diferentes clientes

## Arquitectura de alto nivel

### Componentes del Control Plane (Master)
- **API Server:** Punto de entrada para todas las operaciones del cluster
- **etcd:** Base de datos distribuida que almacena el estado del cluster
- **Scheduler:** Asigna Pods a nodos según recursos disponibles
- **Controller Manager:** Ejecuta controladores que regulan el estado del cluster
- **Cloud Controller Manager:** Integración con proveedores cloud (opcional)

### Componentes de los Worker Nodes
- **Kubelet:** Agente que ejecuta en cada nodo y gestiona los Pods
- **Container Runtime:** Motor de contenedores (Docker, containerd, CRI-O)
- **Kube-proxy:** Gestiona las reglas de red y balanceo de carga

### Objetos fundamentales
- **Pod:** Unidad mínima desplegable, agrupa uno o más contenedores
- **Service:** Abstracción para exponer aplicaciones como servicios de red
- **Volume:** Almacenamiento persistente para los Pods
- **Namespace:** Aislamiento lógico de recursos dentro del cluster

## Diagrama de arquitectura detallado (ASCII)

```
┌─────────────────────────────────────────────────────┐
│            CONTROL PLANE (Master)                   │
├─────────────────────────────────────────────────────┤
│  API Server  │  Scheduler  │  Controller Manager   │
│              │             │                        │
│              └─────────────┴────────────────────────│
│                      etcd                           │
└─────────────────────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
┌───────▼──────┐ ┌──────▼──────┐ ┌─────▼───────┐
│  Worker 1    │ │  Worker 2   │ │  Worker 3   │
├──────────────┤ ├─────────────┤ ├─────────────┤
│  Kubelet     │ │  Kubelet    │ │  Kubelet    │
│  Kube-proxy  │ │  Kube-proxy │ │  Kube-proxy │
│  Container   │ │  Container  │ │  Container  │
│  Runtime     │ │  Runtime    │ │  Runtime    │
├──────────────┤ ├─────────────┤ ├─────────────┤
│ Pod │ Pod    │ │ Pod │ Pod   │ │ Pod │ Pod   │
└──────────────┘ └─────────────┘ └─────────────┘
```

## Flujo de trabajo típico
1. **Desarrollador:** Escribe manifiestos YAML con la configuración deseada
2. **kubectl:** Envía los manifiestos al API Server
3. **API Server:** Valida y almacena en etcd
4. **Scheduler:** Decide en qué nodo ejecutar cada Pod
5. **Kubelet:** Recibe instrucciones y lanza los contenedores
6. **Kube-proxy:** Configura reglas de red para acceso a servicios

## Kubernetes vs Docker Compose

| Característica | Docker Compose | Kubernetes |
|----------------|----------------|------------|
| Complejidad | Baja | Media-Alta |
| Escalabilidad | Limitada | Muy alta |
| Auto-reparación | No | Sí |
| Multi-host | No (solo local) | Sí |
| Balanceo de carga | Básico | Avanzado |
| Actualizaciones | Manual | Automatizadas |
| Ecosistema | Pequeño | Muy extenso |
| Uso ideal | Desarrollo local | Producción a escala |

## Distribuciones y sabores de Kubernetes

### Para desarrollo local
- **Minikube:** Cluster de un solo nodo, fácil de instalar
- **Kind:** Kubernetes in Docker, rápido y ligero
- **k3s:** Versión ligera de K8s, ideal para edge computing
- **Docker Desktop:** Incluye K8s integrado (Windows/Mac)

### Para producción
- **Kubernetes vanilla:** Instalación estándar oficial
- **OpenShift:** Distribución empresarial de Red Hat
- **Rancher:** Plataforma de gestión multi-cluster
- **Cloud providers:** EKS (AWS), GKE (Google), AKS (Azure)

## Contexto actual y adopción
Kubernetes es el estándar de facto para la orquestación de contenedores:
- **Más del 90%** de empresas Fortune 500 usan Kubernetes
- **Ecosistema CNCF:** Más de 150 proyectos certificados
- **Comunidad:** +3000 contribuidores activos
- **Casos de éxito:** Netflix, Spotify, Airbnb, The New York Times, Pokémon GO

## ¿Cuándo NO usar Kubernetes?
- Aplicaciones monolíticas simples
- Equipos pequeños sin experiencia en DevOps
- Proyectos con recursos limitados
- Cuando Docker Compose es suficiente
- Aplicaciones sin requisitos de escalabilidad
