# Ejercicio Final - CI/CD Completo

## Objetivo

Implementar un **pipeline CI/CD end-to-end completo** que demuestre dominio de todos los conceptos del tema.

---

## Proyecto: E-Commerce API

Crear una API REST de e-commerce con:
- Autenticación
- Gestión de productos
- Carrito de compras
- Sistema de pedidos

---

## Requisitos del Pipeline

### ✅ 1. Integración Continua (CI)

- [ ] **Linting**: ESLint, Prettier
- [ ] **Tests unitarios**: Jest (>80% coverage)
- [ ] **Tests integración**: Supertest + DB test
- [ ] **Tests E2E**: Playwright
- [ ] **Security scanning**: Snyk, TruffleHog
- [ ] **Build**: Docker image
- [ ] **Matrix testing**: Node 16, 18, 20

### ✅ 2. Entrega Continua (CD)

- [ ] **Deploy staging**: Automático en push a `develop`
- [ ] **Deploy production**: Manual approval en push a `main`
- [ ] **Preview deployments**: Por cada PR
- [ ] **Rollback automático**: Si health checks fallan

### ✅ 3. Gestión de Secretos

- [ ] Environment-based secrets (staging/prod)
- [ ] No secrets en código
- [ ] Secret scanning habilitado

### ✅ 4. Pruebas Post-Despliegue

- [ ] Smoke tests
- [ ] Health checks con reintentos
- [ ] Integration tests contra staging/prod
- [ ] Performance tests con k6

### ✅ 5. Estrategia de Despliegue

Implementar **UNA** de estas:
- [ ] Blue-Green deployment
- [ ] Canary deployment (10% → 50% → 100%)
- [ ] Rolling update con readiness probes

### ✅ 6. Monitorización

- [ ] Prometheus metrics expuestos
- [ ] DORA metrics calculadas
- [ ] Dashboard de métricas
- [ ] Reports semanales automáticos
- [ ] Alertas configuradas

---

## Estructura del Proyecto

```
ecommerce-api/
├── src/
│   ├── routes/
│   ├── controllers/
│   ├── models/
│   ├── middleware/
│   └── app.js
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── cd-staging.yml
│       ├── cd-production.yml
│       └── metrics.yml
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── scripts/
│   ├── smoke-tests.sh
│   ├── rollback.sh
│   └── metrics.sh
├── Dockerfile
├── docker-compose.yml
├── package.json
└── README.md
```

---

## Implementación Paso a Paso

### Fase 1: Aplicación Base (Día 1-2)

**API Endpoints requeridos:**

```javascript
// Auth
POST /auth/register
POST /auth/login
POST /auth/logout

// Products
GET /api/products
GET /api/products/:id
POST /api/products (admin)
PUT /api/products/:id (admin)
DELETE /api/products/:id (admin)

// Cart
GET /api/cart
POST /api/cart/items
DELETE /api/cart/items/:id

// Orders
GET /api/orders
POST /api/orders
GET /api/orders/:id

// Health
GET /health
GET /metrics
```

**Tecnologías:**
- Express.js
- PostgreSQL
- JWT para auth
- Docker

### Fase 2: Tests (Día 3)

**Unit Tests (>80% coverage):**
```javascript
// tests/unit/product.test.js
describe('Product Model', () => {
  test('should validate product data', () => {
    const product = new Product({
      name: 'Laptop',
      price: 999.99,
      stock: 10
    });
    expect(product.validate()).toBe(true);
  });
});
```

**Integration Tests:**
```javascript
// tests/integration/api.test.js
describe('Products API', () => {
  test('GET /api/products returns products', async () => {
    const res = await request(app).get('/api/products');
    expect(res.status).toBe(200);
    expect(Array.isArray(res.body)).toBe(true);
  });
});
```

**E2E Tests:**
```javascript
// tests/e2e/checkout.spec.js
test('Complete checkout flow', async ({ page }) => {
  await page.goto('/');
  await page.click('text=Add to Cart');
  await page.click('text=Checkout');
  await expect(page.locator('h1')).toContainText('Order Confirmed');
});
```

### Fase 3: Pipeline CI (Día 4)

**.github/workflows/ci.yml:**

```yaml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run lint

  test-unit:
    needs: lint
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [16, 18, 20]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm run test:unit -- --coverage
      - uses: codecov/codecov-action@v3

  test-integration:
    needs: lint
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: testpass
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run test:integration

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  build:
    needs: [test-unit, test-integration]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: myapp:${{ github.sha }}
```

### Fase 4: Deploy Staging (Día 5)

**.github/workflows/cd-staging.yml:**

```yaml
name: Deploy Staging

on:
  push:
    branches: [develop]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.ecommerce-api.com
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to staging
        run: ./deploy.sh staging
      
      - name: Smoke tests
        run: ./scripts/smoke-tests.sh https://staging.ecommerce-api.com
      
      - name: Integration tests
        env:
          BASE_URL: https://staging.ecommerce-api.com
        run: npm run test:integration:remote
      
      - name: Performance tests
        run: docker run --rm -i grafana/k6 run - < load-test.js
```

### Fase 5: Deploy Production (Día 6)

**.github/workflows/cd-production.yml:**

```yaml
name: Deploy Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://ecommerce-api.com
    outputs:
      previous-version: ${{ steps.backup.outputs.version }}
    steps:
      - uses: actions/checkout@v3
      
      - name: Backup current version
        id: backup
        run: |
          CURRENT=$(kubectl get deployment api -o jsonpath='{.spec.template.spec.containers[0].image}')
          echo "version=$CURRENT" >> $GITHUB_OUTPUT
      
      - name: Deploy new version
        run: kubectl set image deployment/api api=myapp:${{ github.sha }}
      
      - name: Wait for rollout
        run: kubectl rollout status deployment/api --timeout=5m
  
  post-deploy-tests:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Health checks
        id: health
        run: ./scripts/smoke-tests.sh https://ecommerce-api.com
        continue-on-error: true
      
      - name: E2E critical paths
        run: npx playwright test --grep "@critical"
        continue-on-error: true
  
  rollback:
    needs: [deploy, post-deploy-tests]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: Rollback
        run: |
          kubectl set image deployment/api \
            api=${{ needs.deploy.outputs.previous-version }}
          kubectl rollout status deployment/api
      
      - name: Alert team
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          text: '🚨 Production deployment failed and rolled back'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Fase 6: Monitorización (Día 7)

**Prometheus metrics en la app:**

```javascript
// src/middleware/metrics.js
const promClient = require('prom-client');
const register = new promClient.Registry();

const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests',
  labelNames: ['method', 'route', 'status']
});

const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status']
});

const ordersTotal = new promClient.Counter({
  name: 'orders_total',
  help: 'Total orders created',
  labelNames: ['status']
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(ordersTotal);

module.exports = { register, httpRequestDuration, httpRequestTotal, ordersTotal };
```

**Weekly metrics report:**

**.github/workflows/metrics.yml:**

```yaml
name: Weekly Metrics Report

on:
  schedule:
    - cron: '0 9 * * 1'

jobs:
  metrics:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Calculate DORA metrics
        run: ./scripts/metrics.sh > report.md
      
      - name: Send report
        uses: 8398a7/action-slack@v3
        with:
          status: custom
          custom_payload: |
            {
              text: "📊 Weekly CI/CD Metrics",
              attachments: [{
                color: 'good',
                text: '${{ steps.report.outputs.content }}'
              }]
            }
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Entregables

### 1. Repositorio GitHub

- Código completo
- Todos los workflows funcionando
- README con instrucciones

### 2. Documentación

**PIPELINE.md:**
```markdown
# Pipeline CI/CD Documentation

## Architecture

[Diagrama del pipeline]

## Workflows

### CI Pipeline
- Triggers: Push, PR
- Steps: Lint → Test → Build
- Duration: 5-7 minutes

### CD Staging
- Trigger: Push to develop
- Auto-deploy + tests
- Duration: 3-4 minutes

### CD Production
- Trigger: Push to main
- Manual approval required
- Rollback automático si falla
- Duration: 5-6 minutes

## Metrics

- Deployment Frequency: X/day
- Lead Time: X minutes
- MTTR: X minutes
- Change Failure Rate: X%

## Runbooks

### Deployment Manual
[Pasos para deploy manual]

### Rollback
[Cómo hacer rollback]

### Troubleshooting
[Problemas comunes]
```

### 3. Video Demo (Opcional)

- 5-10 minutos
- Mostrar:
  - Commit → Pipeline execution
  - Deploy automático
  - Tests post-deploy
  - Métricas dashboard

---

## Criterios de Evaluación

| Criterio | Puntos | Descripción |
|----------|--------|-------------|
| **CI Pipeline** | 20 | Lint, tests, build funcionando |
| **CD Pipeline** | 20 | Deploy automático staging + prod |
| **Secretos** | 10 | Gestión segura implementada |
| **Post-Deploy Tests** | 15 | Smoke + health + integration |
| **Estrategia Deploy** | 15 | Blue-Green o Canary o Rolling |
| **Monitorización** | 10 | Métricas + dashboard + reports |
| **Documentación** | 10 | README + PIPELINE.md completos |
| **Total** | **100** | |

---

## Bonus (+20 puntos)

- [ ] **Multi-region deploy** (+5)
- [ ] **Feature flags** (+5)
- [ ] **Chaos engineering** (+5)
- [ ] **Cost optimization** (+5)

---

## Recursos

- [Código de ejemplo](../ejemplos/)
- [Templates de workflows](../.github/workflows/)
- [Scripts útiles](../scripts/)

---

**¡Demuestra todo lo aprendido! 🚀**

**Tiempo estimado:** 7 días
**Dificultad:** Alta
**Impacto en aprendizaje:** Muy Alto
