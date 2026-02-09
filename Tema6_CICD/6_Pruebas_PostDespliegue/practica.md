# Prácticas - Pruebas Post-Despliegue

## Práctica 1: Smoke Tests Básicos

**smoke-tests.sh:**
```bash
#!/bin/bash
set -e

URL="${1:-http://localhost:3000}"
echo "🔥 Running smoke tests against $URL"

# Test 1: Health
curl -f "$URL/health" || exit 1
echo "✅ Health OK"

# Test 2: API
curl -f "$URL/api/users" || exit 1
echo "✅ API OK"

echo "✅ All smoke tests passed!"
```

**Workflow:**
```yaml
- name: Smoke tests
  run: |
    chmod +x smoke-tests.sh
    ./smoke-tests.sh https://myapp.com
```

---

## Práctica 2: Integration Tests con Jest

**post-deploy.test.js:**
```javascript
const BASE_URL = process.env.BASE_URL || 'http://localhost:3000';

describe('Post-Deployment Tests', () => {
  test('API is accessible', async () => {
    const res = await fetch(`${BASE_URL}/health`);
    expect(res.ok).toBe(true);
  });

  test('Can create and fetch data', async () => {
    const create = await fetch(`${BASE_URL}/api/users`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: 'Test', email: 'test@test.com' })
    });
    expect(create.ok).toBe(true);
    
    const data = await create.json();
    expect(data).toHaveProperty('id');
  });
});
```

**package.json:**
```json
{
  "scripts": {
    "test:post-deploy": "jest post-deploy.test.js"
  }
}
```

**Workflow:**
```yaml
- name: Post-deployment tests
  env:
    BASE_URL: https://staging.myapp.com
  run: npm run test:post-deploy
```

---

## Práctica 3: E2E con Playwright

**e2e/production.spec.js:**
```javascript
const { test, expect } = require('@playwright/test');

test('User can login and view dashboard', async ({ page }) => {
  await page.goto('https://myapp.com');
  
  await page.click('text=Login');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL(/.*dashboard/);
  await expect(page.locator('h1')).toContainText('Dashboard');
});
```

**Workflow:**
```yaml
- name: E2E tests
  run: |
    npx playwright install chromium --with-deps
    npx playwright test e2e/production.spec.js
```

---

## Práctica 4: Load Testing con k6

**load-test.js:**
```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 10,
  duration: '30s',
  thresholds: {
    http_req_duration: ['p(95)<500'],
  },
};

export default function () {
  const res = http.get('https://myapp.com/api/users');
  check(res, { 'status 200': (r) => r.status === 200 });
}
```

**Workflow:**
```yaml
- name: Load test
  run: |
    docker run --rm -i grafana/k6 run - < load-test.js
```

---

## Práctica 5: Rollback Automático

```yaml
name: Deploy with Auto-Rollback

jobs:
  deploy:
    outputs:
      previous-version: ${{ steps.backup.outputs.version }}
    steps:
      - id: backup
        run: echo "version=v1.2.3" >> $GITHUB_OUTPUT
      - run: ./deploy.sh

  test:
    needs: deploy
    steps:
      - run: ./smoke-tests.sh
    continue-on-error: true

  rollback:
    needs: [deploy, test]
    if: failure()
    steps:
      - run: |
          echo "Rolling back to ${{ needs.deploy.outputs.previous-version }}"
          ./rollback.sh ${{ needs.deploy.outputs.previous-version }}
```

---

## Ejercicios

1. **Implementa health check avanzado** con verificación de DB, Redis, API externa
2. **Crea suite completa** de smoke tests para tu aplicación
3. **Añade performance budget** que falle si response time > 500ms
4. **Implementa synthetic monitoring** cada 5 minutos

---

## Checklist

- [ ] Smoke tests implementados
- [ ] Health check endpoint creado
- [ ] Integration tests ejecutándose
- [ ] E2E tests críticos cubiertos
- [ ] Load tests configurados
- [ ] Rollback automático funcionando
- [ ] Alertas configuradas

**¡Tests post-despliegue dominados! ✅**
