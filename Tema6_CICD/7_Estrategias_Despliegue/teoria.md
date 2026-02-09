# Estrategias de Despliegue

## Introducción

Las **estrategias de despliegue** determinan cómo se realiza la transición de una versión de la aplicación a otra en producción.

---

## 1. Recreate (Recrear)

### Descripción
Detener completamente la versión antigua antes de iniciar la nueva.

### Proceso

```
v1 Running → Stop v1 → DOWNTIME → Start v2 → v2 Running
```

### Implementación

```yaml
# Docker Compose
docker-compose down
docker-compose up -d  # Nueva versión
```

### Ventajas
- ✅ Simple de implementar
- ✅ No requiere recursos extra
- ✅ No hay versiones mezcladas

### Desventajas
- ❌ Downtime inevitable
- ❌ No puede revertir rápidamente
- ❌ No recomendado para producción

### Cuándo Usar
- Entornos de desarrollo
- Aplicaciones no críticas
- Ventanas de mantenimiento programadas

---

## 2. Rolling Update (Actualización Progresiva)

### Descripción
Actualizar instancias gradualmente, una a una o en grupos.

### Proceso

```
[v1] [v1] [v1] [v1]
  ↓
[v2] [v1] [v1] [v1]
  ↓
[v2] [v2] [v1] [v1]
  ↓
[v2] [v2] [v2] [v1]
  ↓
[v2] [v2] [v2] [v2]
```

### Implementación Kubernetes

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Máximo 1 pod extra durante update
      maxUnavailable: 1  # Máximo 1 pod no disponible
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:v2
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

**Deploy:**
```bash
kubectl apply -f deployment.yaml
kubectl rollout status deployment/myapp
```

### Ventajas
- ✅ Zero downtime
- ✅ Gradual, menos riesgo
- ✅ Fácil rollback

### Desventajas
- ❌ Dos versiones simultáneas
- ❌ Más lento que recreate
- ❌ Requiere backward compatibility

### Cuándo Usar
- Aplicaciones web stateless
- Microservicios
- APIs REST

---

## 3. Blue-Green Deployment

### Descripción
Mantener dos entornos idénticos (Blue y Green). Cambiar tráfico entre ellos.

### Proceso

```
Blue (v1) ← 100% traffic
Green (v2) ← 0% traffic (deploy nueva versión)
    ↓
Testing Green
    ↓
Blue (v1) ← 0% traffic
Green (v2) ← 100% traffic (switch)
```

### Implementación

**Con Load Balancer:**

```yaml
# Inicial: Blue activo
Load Balancer → Blue Environment (v1)
                Green Environment (idle)

# Deploy a Green
Load Balancer → Blue Environment (v1)
                Green Environment (v2) ← Deploy here

# Switch traffic
Load Balancer → Green Environment (v2)
                Blue Environment (v1) ← Keep for rollback

# Si OK, actualizar Blue para próximo deploy
```

**Con Kubernetes Service:**

```yaml
# Service apuntando a Blue
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
    version: blue  # ← Cambia a "green" para switch
  ports:
  - port: 80
    targetPort: 3000
```

**Script de switch:**

```bash
#!/bin/bash
# blue-green-switch.sh

CURRENT=$(kubectl get service myapp -o jsonpath='{.spec.selector.version}')

if [ "$CURRENT" == "blue" ]; then
  NEW="green"
else
  NEW="blue"
fi

echo "Switching from $CURRENT to $NEW..."
kubectl patch service myapp -p "{\"spec\":{\"selector\":{\"version\":\"$NEW\"}}}"
echo "✅ Traffic now on $NEW"
```

### Ventajas
- ✅ Zero downtime
- ✅ Rollback instantáneo
- ✅ Testing en producción antes de switch
- ✅ Ambiente completo para pruebas

### Desventajas
- ❌ Requiere 2x recursos
- ❌ Database migrations complejas
- ❌ Más infraestructura

### Cuándo Usar
- Aplicaciones críticas
- Cuando puedes permitir 2x recursos
- Necesitas rollback inmediato

---

## 4. Canary Deployment

### Descripción
Liberar nueva versión a un pequeño % de usuarios primero, luego gradualmente aumentar.

### Proceso

```
v1: 100% → v1: 90% + v2: 10%
         → v1: 75% + v2: 25%
         → v1: 50% + v2: 50%
         → v1: 0%  + v2: 100%
```

### Implementación con Istio

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp.example.com
  http:
  - match:
    - headers:
        user-type:
          exact: "beta-tester"
    route:
    - destination:
        host: myapp
        subset: v2
      weight: 100
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 90
    - destination:
        host: myapp
        subset: v2
      weight: 10  # 10% de usuarios a v2
```

### Implementación Manual (Nginx)

```nginx
upstream backend {
    server app-v1:3000 weight=90;
    server app-v2:3000 weight=10;  # 10% canary
}
```

### Ventajas
- ✅ Riesgo minimizado
- ✅ Testing real con usuarios
- ✅ Feedback temprano
- ✅ Rollback parcial fácil

### Desventajas
- ❌ Requiere service mesh o load balancer avanzado
- ❌ Monitoring complejo
- ❌ Dos versiones en producción

### Cuándo Usar
- Releases de alto riesgo
- Nuevas features experimentales
- Aplicaciones con muchos usuarios

---

## 5. A/B Testing Deployment

### Descripción
Similar a Canary pero basado en reglas de negocio, no solo porcentaje.

### Proceso

```
Users → Router → v1 (users with feature flag OFF)
               → v2 (users with feature flag ON)
```

### Implementación

```yaml
# Feature flag routing
if (user.hasFeature('new-checkout')) {
  route to v2
} else {
  route to v1
}
```

### Ventajas
- ✅ Control granular por usuario
- ✅ Testing de features específicas
- ✅ Datos para decisiones de negocio

### Desventajas
- ❌ Requiere feature flag system
- ❌ Complejidad en código
- ❌ Análisis de datos necesario

---

## 6. Shadow Deployment

### Descripción
Enviar tráfico real a nueva versión pero ignorar sus respuestas (solo para testing).

### Proceso

```
Request → v1 → Response to user
        ↘ v2 → Response discarded (only logs/metrics)
```

### Implementación con Istio

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  http:
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 100
    mirror:
      host: myapp
      subset: v2  # Shadow traffic
```

### Ventajas
- ✅ Testing con tráfico real
- ✅ Zero riesgo para usuarios
- ✅ Detectar bugs antes de release

### Desventajas
- ❌ 2x carga en backend
- ❌ Requiere service mesh
- ❌ Side effects en v2 (DB writes, etc.)

---

## Comparativa de Estrategias

| Estrategia | Downtime | Complejidad | Recursos | Rollback | Riesgo |
|------------|----------|-------------|----------|----------|--------|
| **Recreate** | ❌ Sí | 🟢 Baja | 🟢 1x | 🔴 Lento | 🔴 Alto |
| **Rolling** | ✅ No | 🟡 Media | 🟡 1.2x | 🟡 Medio | 🟡 Medio |
| **Blue-Green** | ✅ No | 🟡 Media | 🔴 2x | 🟢 Rápido | 🟢 Bajo |
| **Canary** | ✅ No | 🔴 Alta | 🟡 1.2x | 🟢 Rápido | 🟢 Muy bajo |
| **A/B Testing** | ✅ No | 🔴 Alta | 🟡 1.2x | 🟢 Rápido | 🟢 Bajo |
| **Shadow** | ✅ No | 🔴 Alta | 🔴 2x | N/A | 🟢 Zero |

---

## Decisión: ¿Qué Estrategia Elegir?

```
¿Puedes tener downtime?
  └─ Sí → Recreate
  └─ No ↓

¿Tienes 2x recursos?
  └─ Sí → Blue-Green (rollback rápido) o Shadow (testing)
  └─ No ↓

¿Tienes service mesh (Istio/Linkerd)?
  └─ Sí → Canary (mínimo riesgo)
  └─ No ↓

¿Necesitas backward compatibility?
  └─ Sí → Rolling Update
  └─ No → Blue-Green o espera para service mesh
```

---

## Best Practices

1. **Siempre con health checks** - Verificar antes de enviar tráfico
2. **Automated rollback** - Si métricas degradan, revertir
3. **Gradual rollout** - No 0% → 100% de golpe
4. **Monitor everything** - Métricas en tiempo real
5. **Database migrations** - Backward compatible siempre
6. **Feature flags** - Desacoplar deploy de feature release

---

**Próxima sección:** Monitorización y Evaluación
