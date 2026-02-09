# Prácticas - Estrategias de Despliegue

## Práctica 1: Rolling Update con Docker Compose

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  app:
    image: myapp:${VERSION:-latest}
    deploy:
      replicas: 4
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      rollback_config:
        parallelism: 1
        delay: 5s
    ports:
      - "3000:3000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 3s
      retries: 3
```

**Deploy:**
```bash
# Deploy v2
VERSION=v2 docker-compose up -d

# Ver progreso
docker-compose ps

# Rollback si necesario
docker-compose rollback
```

---

## Práctica 2: Blue-Green con Scripts

**Estructura:**
```
/app
  /blue  (v1)
  /green (v2)
nginx.conf
```

**nginx.conf:**
```nginx
upstream app {
    server localhost:3001;  # blue o green
}

server {
    listen 80;
    location / {
        proxy_pass http://app;
    }
}
```

**blue-green-deploy.sh:**
```bash
#!/bin/bash
set -e

NEW_VERSION=$1
CURRENT=$(cat /tmp/current-env)  # "blue" o "green"

if [ "$CURRENT" == "blue" ]; then
    NEW_ENV="green"
    NEW_PORT=3002
    OLD_PORT=3001
else
    NEW_ENV="blue"
    NEW_PORT=3001
    OLD_PORT=3002
fi

echo "Deploying to $NEW_ENV..."

# 1. Deploy nueva versión al entorno inactivo
cd /app/$NEW_ENV
git pull origin main
npm install
PORT=$NEW_PORT npm start &

# 2. Esperar que esté listo
sleep 10
curl -f http://localhost:$NEW_PORT/health || exit 1

# 3. Switch nginx
sed -i "s/localhost:$OLD_PORT/localhost:$NEW_PORT/" /etc/nginx/nginx.conf
nginx -s reload

# 4. Guardar estado
echo "$NEW_ENV" > /tmp/current-env

echo "✅ Deployed to $NEW_ENV on port $NEW_PORT"
```

---

## Práctica 3: Canary con Nginx

**nginx-canary.conf:**
```nginx
# 90% a v1, 10% a v2
upstream backend_v1 {
    server app-v1:3000;
}

upstream backend_v2 {
    server app-v2:3000;
}

split_clients "${remote_addr}" $backend_pool {
    10%     v2;  # 10% canary
    *       v1;  # 90% stable
}

server {
    listen 80;
    
    location / {
        proxy_pass http://backend_$backend_pool;
    }
}
```

**Incrementar canary:**
```nginx
split_clients "${remote_addr}" $backend_pool {
    50%     v2;  # Ahora 50%
    *       v1;
}
```

---

## Práctica 4: Feature Flags

**app.js:**
```javascript
const unleash = require('unleash-client');

unleash.initialize({
  url: 'http://unleash.example.com/api/',
  appName: 'my-app',
  instanceId: process.env.HOSTNAME
});

app.get('/feature', (req, res) => {
  if (unleash.isEnabled('new-feature', { userId: req.user.id })) {
    // Nueva versión
    res.json({ version: 'v2', feature: 'new' });
  } else {
    // Versión antigua
    res.json({ version: 'v1', feature: 'old' });
  }
});
```

---

## Práctica 5: Kubernetes Rolling Update

**deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
      - name: myapp
        image: myapp:v2
        ports:
        - containerPort: 3000
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 15
          periodSeconds: 20
```

**Deploy y monitorizar:**
```bash
# Deploy
kubectl apply -f deployment.yaml

# Ver progreso en tiempo real
kubectl rollout status deployment/myapp

# Si algo falla, rollback
kubectl rollout undo deployment/myapp

# Ver historial
kubectl rollout history deployment/myapp
```

---

## Ejercicios

### Ejercicio 1: Blue-Green Completo
Implementa blue-green deployment completo con health checks y rollback automático.

### Ejercicio 2: Canary Progresivo
Crea un script que incremente canary de 10% → 25% → 50% → 100% automáticamente.

### Ejercicio 3: A/B Testing
Implementa A/B testing basado en cookies de usuario.

### Ejercicio 4: Comparar Estrategias
Mide tiempo de deploy y riesgo para cada estrategia en tu aplicación.

---

## Checklist

- [ ] Rolling update implementado
- [ ] Blue-green deployment probado
- [ ] Canary deployment configurado
- [ ] Feature flags integrados
- [ ] Rollback automático funcionando
- [ ] Health checks en todas las estrategias
- [ ] Métricas comparativas documentadas

**¡Estrategias de despliegue dominadas! 🚀**
