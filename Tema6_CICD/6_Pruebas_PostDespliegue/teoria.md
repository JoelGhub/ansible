# Pruebas Post-Despliegue

## Introducción

Las **pruebas post-despliegue** verifican que la aplicación funciona correctamente en el entorno destino DESPUÉS del deployment.

### ¿Por Qué Son Importantes?

- ✅ Detectar problemas específicos del entorno
- ✅ Validar configuración de producción
- ✅ Verificar integraciones con servicios externos
- ✅ Asegurar que el deployment fue exitoso
- ✅ Permitir rollback rápido si hay problemas

---

## Tipos de Pruebas Post-Despliegue

### 1. Smoke Tests

**Objetivo:** Verificar funcionalidad básica crítica.

**Características:**
- Rápidos (< 5 minutos)
- Prueban happy paths
- Críticos para la aplicación
- Se ejecutan inmediatamente después del deploy

**Ejemplo:**

```bash
#!/bin/bash
# smoke-tests.sh

BASE_URL="https://api.example.com"

echo "🔥 Running smoke tests..."

# Test 1: Health check
echo "Testing /health..."
if ! curl -f -s "$BASE_URL/health" | grep -q "healthy"; then
  echo "❌ Health check failed"
  exit 1
fi
echo "✅ Health check passed"

# Test 2: Login endpoint
echo "Testing /auth/login..."
response=$(curl -s -X POST "$BASE_URL/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test123"}')

if ! echo "$response" | grep -q "token"; then
  echo "❌ Login failed"
  exit 1
fi
echo "✅ Login passed"

# Test 3: Database connectivity
echo "Testing /api/users..."
if ! curl -f -s "$BASE_URL/api/users" > /dev/null; then
  echo "❌ Database connectivity failed"
  exit 1
fi
echo "✅ Database connectivity passed"

echo "✅ All smoke tests passed!"
```

**En CI/CD:**

```yaml
- name: Run smoke tests
  run: |
    sleep 30  # Wait for service to be ready
    chmod +x smoke-tests.sh
    ./smoke-tests.sh
```

---

### 2. Health Checks

**Objetivo:** Verificar que el servicio está vivo y respondiendo.

**Endpoint típico:**

```javascript
// Express.js
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    uptime: process.uptime(),
    timestamp: new Date().toISOString(),
    version: process.env.VERSION || '1.0.0'
  });
});
```

**Health check avanzado:**

```javascript
app.get('/health', async (req, res) => {
  const checks = {
    database: false,
    redis: false,
    externalAPI: false
  };

  try {
    // Check database
    await db.query('SELECT 1');
    checks.database = true;

    // Check Redis
    await redis.ping();
    checks.redis = true;

    // Check external API
    const response = await fetch('https://external-api.com/health');
    checks.externalAPI = response.ok;

    const allHealthy = Object.values(checks).every(c => c === true);
    
    res.status(allHealthy ? 200 : 503).json({
      status: allHealthy ? 'healthy' : 'degraded',
      checks,
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      checks,
      error: error.message
    });
  }
});
```

**En CI/CD:**

```yaml
- name: Health check with retries
  run: |
    for i in {1..10}; do
      if curl -f https://api.example.com/health; then
        echo "✅ Service is healthy"
        exit 0
      fi
      echo "Attempt $i/10 failed, waiting..."
      sleep 10
    done
    echo "❌ Health check failed after 10 attempts"
    exit 1
```

---

### 3. Integration Tests

**Objetivo:** Verificar que los componentes trabajan juntos correctamente.

**Ejemplo con Supertest:**

```javascript
// tests/integration/api.test.js
const request = require('supertest');

const BASE_URL = process.env.BASE_URL || 'http://localhost:3000';

describe('API Integration Tests (Post-Deployment)', () => {
  test('Should create user via API', async () => {
    const response = await request(BASE_URL)
      .post('/api/users')
      .send({ name: 'John Doe', email: 'john@test.com' })
      .expect(201);
    
    expect(response.body).toHaveProperty('id');
    expect(response.body.name).toBe('John Doe');
  });

  test('Should retrieve users from database', async () => {
    const response = await request(BASE_URL)
      .get('/api/users')
      .expect(200);
    
    expect(Array.isArray(response.body)).toBe(true);
  });

  test('Should handle authentication', async () => {
    const response = await request(BASE_URL)
      .post('/auth/login')
      .send({ email: 'test@example.com', password: 'password' })
      .expect(200);
    
    expect(response.body).toHaveProperty('token');
  });
});
```

**En CI/CD:**

```yaml
- name: Run integration tests
  env:
    BASE_URL: https://staging.example.com
  run: |
    npm run test:integration
```

---

### 4. End-to-End Tests

**Objetivo:** Simular flujos completos de usuario.

**Ejemplo con Playwright:**

```javascript
// e2e/checkout.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Checkout Flow (Production)', () => {
  test('Complete purchase flow', async ({ page }) => {
    // 1. Login
    await page.goto('https://myapp.com/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'testpass');
    await page.click('button[type="submit"]');
    await expect(page).toHaveURL('https://myapp.com/dashboard');

    // 2. Add to cart
    await page.goto('https://myapp.com/products');
    await page.click('text=Add to Cart >> nth=0');
    await expect(page.locator('.cart-count')).toHaveText('1');

    // 3. Checkout
    await page.click('text=Cart');
    await page.click('text=Checkout');
    
    // 4. Verify order confirmation
    await expect(page.locator('h1')).toContainText('Order Confirmed');
  });
});
```

**En CI/CD:**

```yaml
- name: E2E tests on production
  if: github.ref == 'refs/heads/main'
  run: |
    npx playwright install --with-deps
    npx playwright test --grep "Production"
```

---

### 5. Performance Tests

**Objetivo:** Verificar que el despliegue no degradó el performance.

**Ejemplo con k6:**

```javascript
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 50 },   // Ramp up
    { duration: '3m', target: 50 },   // Stay
    { duration: '1m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% < 500ms
    http_req_failed: ['rate<0.01'],   // Error rate < 1%
  },
};

export default function () {
  const res = http.get('https://api.example.com/users');
  
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  sleep(1);
}
```

**En CI/CD:**

```yaml
- name: Performance tests
  run: |
    docker run --rm -i grafana/k6 run - < load-test.js
```

---

## Implementación en Pipeline

### Pipeline Completo con Post-Deployment Tests

```yaml
name: Deploy with Post-Tests

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    outputs:
      deployment-url: ${{ steps.deploy.outputs.url }}
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy
        id: deploy
        run: |
          ./deploy.sh
          echo "url=https://api.example.com" >> $GITHUB_OUTPUT
  
  # POST-DEPLOYMENT TESTS
  smoke-tests:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Wait for deployment
        run: sleep 30
      
      - name: Smoke tests
        run: ./smoke-tests.sh
  
  integration-tests:
    needs: smoke-tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run integration tests
        env:
          BASE_URL: ${{ needs.deploy.outputs.deployment-url }}
        run: npm run test:integration
  
  e2e-tests:
    needs: smoke-tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run E2E tests
        run: |
          npx playwright install --with-deps
          npx playwright test
  
  performance-tests:
    needs: [smoke-tests, integration-tests]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Load test
        run: docker run --rm -i grafana/k6 run - < load-test.js
  
  # ROLLBACK SI ALGUNO FALLA
  rollback:
    needs: [smoke-tests, integration-tests, e2e-tests]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: Rollback deployment
        run: |
          echo "❌ Tests failed, rolling back..."
          ./rollback.sh
```

---

## Monitoring y Alerting

### Prometheus + Grafana

**Métricas a monitorizar:**

```yaml
Application Metrics:
  - Request rate
  - Error rate
  - Response time (p50, p95, p99)
  - Active connections

System Metrics:
  - CPU usage
  - Memory usage
  - Disk I/O
  - Network traffic

Business Metrics:
  - Successful transactions
  - User signups
  - Revenue
```

### Synthetic Monitoring

**Monitoreo continuo post-deployment:**

```yaml
name: Continuous Monitoring

on:
  schedule:
    - cron: '*/5 * * * *'  # Every 5 minutes

jobs:
  monitor:
    runs-on: ubuntu-latest
    steps:
      - name: Health check
        run: |
          if ! curl -f https://api.example.com/health; then
            # Trigger alert
            curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
              -d '{"text":"🚨 Production API is down!"}'
            exit 1
          fi
```

---

## Checklist Post-Deployment

```yaml
Immediate (0-5 min):
  - [ ] Health check passed
  - [ ] Smoke tests passed
  - [ ] Application accessible

Short-term (5-30 min):
  - [ ] Integration tests passed
  - [ ] E2E critical paths working
  - [ ] Error rate normal
  - [ ] Response times acceptable

Long-term (30+ min):
  - [ ] Performance tests passed
  - [ ] Load handling as expected
  - [ ] No memory leaks
  - [ ] Monitoring shows green
```

---

**Próxima sección:** Estrategias de Despliegue
