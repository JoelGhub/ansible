# Fases CI y CD

## Introducción

Un pipeline CI/CD completo se compone de múltiples fases que transforman el código desde el repositorio hasta producción. Cada fase tiene un propósito específico y contribuye a garantizar la calidad y confiabilidad del software.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         PIPELINE CI/CD                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────┐  ┌───────┐  ┌──────┐  ┌─────────┐  ┌────────────┐   │
│  │  Source  │→ │ Build │→ │ Test │→ │ Package │→ │   Deploy   │   │
│  └──────────┘  └───────┘  └──────┘  └─────────┘  └────────────┘   │
│       ↓            ↓          ↓          ↓              ↓           │
│      CI          CI          CI         CD            CD           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## FASE 1: Source Control (Control de Versiones)

### Descripción

Todo comienza con el control de versiones. Es la base de CI/CD.

### Características

- **Version Control System (VCS)**: Git, SVN, Mercurial
- **Hosting**: GitHub, GitLab, Bitbucket, Azure DevOps
- **Branching Strategy**: Git Flow, GitHub Flow, Trunk-Based Development

### Ejemplo: Git Flow

```
main (producción)
    │
    ├─── develop (desarrollo)
    │       │
    │       ├─── feature/login
    │       │
    │       ├─── feature/payment
    │       │
    │       └─── hotfix/critical-bug
    │
    └─── release/v1.2.0
```

### GitHub Flow (más simple, ideal para CI/CD)

```
main (siempre deployable)
    │
    ├─── feature/add-user-profile
    │    │
    │    └─── PR → Tests → Merge → Deploy
    │
    └─── fix/security-patch
         │
         └─── PR → Tests → Merge → Deploy
```

### Triggers Comunes

```yaml
# Push a main
on:
  push:
    branches: [main]

# Pull Request
on:
  pull_request:
    branches: [main]

# Tags (releases)
on:
  push:
    tags:
      - 'v*'

# Manual
on:
  workflow_dispatch:

# Scheduled
on:
  schedule:
    - cron: '0 2 * * *'
```

### Buenas Prácticas

1. **Commits pequeños y frecuentes**
   ```bash
   ✅ BIEN:
   - "feat: Add user login form"
   - "fix: Resolve null pointer in payment"
   - "test: Add unit tests for calculator"
   
   ❌ MAL:
   - "Updated everything"
   - "WIP"
   - "Fixed stuff"
   ```

2. **Conventional Commits**
   ```
   feat: nueva funcionalidad
   fix: corrección de bug
   docs: documentación
   style: formato, sin cambio de código
   refactor: refactorización
   test: añadir tests
   chore: mantenimiento
   ```

3. **Branch Protection**
   - Require PR reviews
   - Require status checks
   - No direct push to main

---

## FASE 2: Build (Construcción)

### Descripción

El proceso de transformar el código fuente en artefactos ejecutables.

### Objetivos

- ✅ Compilar código (si aplica)
- ✅ Resolver dependencias
- ✅ Transpilar/Bundle (JavaScript/TypeScript)
- ✅ Generar artefactos deployables

### Ejemplos por Tecnología

#### JavaScript/Node.js

```yaml
build:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18.x'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Build
      run: npm run build
    
    - name: Upload artifact
      uses: actions/upload-artifact@v3
      with:
        name: dist
        path: dist/
```

#### Java/Maven

```yaml
build:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Setup Java
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: 'maven'
    
    - name: Build with Maven
      run: mvn clean package -DskipTests
    
    - name: Upload JAR
      uses: actions/upload-artifact@v3
      with:
        name: app-jar
        path: target/*.jar
```

#### Python

```yaml
build:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
        cache: 'pip'
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install build
    
    - name: Build package
      run: python -m build
    
    - name: Upload wheel
      uses: actions/upload-artifact@v3
      with:
        name: python-package
        path: dist/
```

#### Docker

```yaml
build:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Build Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: false
        tags: myapp:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

### Optimización del Build

#### 1. Caché de Dependencias

```yaml
# Node.js - automático con setup-node
- uses: actions/setup-node@v3
  with:
    cache: 'npm'

# Maven - automático con setup-java
- uses: actions/setup-java@v3
  with:
    cache: 'maven'

# Manual cache
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

#### 2. Builds Incrementales

```yaml
# Solo build archivos cambiados
- name: Incremental build
  run: |
    if git diff --name-only HEAD~1 | grep -q '^src/'; then
      npm run build
    else
      echo "No source changes, skipping build"
    fi
```

#### 3. Paralelización

```yaml
build:
  strategy:
    matrix:
      module: [frontend, backend, api]
  steps:
    - name: Build ${{ matrix.module }}
      run: npm run build:${{ matrix.module }}
```

### Build Artifacts

Los **artifacts** son el resultado del build:

```yaml
Tipos comunes:
- JAR/WAR (Java)
- wheel/egg (Python)
- tarball (Node.js)
- binary (Go, Rust, C++)
- Docker image
- ZIP/TAR.GZ
```

#### Ejemplo de Upload/Download de Artifacts

```yaml
# Job 1: Build
build:
  steps:
    - run: npm run build
    - uses: actions/upload-artifact@v3
      with:
        name: build-output
        path: dist/

# Job 2: Deploy (usa el artifact)
deploy:
  needs: build
  steps:
    - uses: actions/download-artifact@v3
      with:
        name: build-output
        path: dist/
    - run: ./deploy.sh
```

---

## FASE 3: Test (Pruebas)

### Descripción

Verificación automatizada de que el código funciona correctamente.

### Pirámide de Testing

```
           /\
          /  \
         / E2E \      ← Pocas, lentas, costosas
        /--------\
       /Integration\ ← Moderadas
      /--------------\
     /   Unit Tests   \ ← Muchas, rápidas, baratas
    /------------------\
```

### Tipos de Tests

#### 1. Unit Tests (Pruebas Unitarias)

**Características:**
- Prueban funciones/métodos individuales
- Aisladas (sin dependencias externas)
- Rápidas (milisegundos)
- Deben ser muchas (80% de tus tests)

**Ejemplo:**

```javascript
// calculator.test.js
test('add two numbers', () => {
  const calc = new Calculator();
  expect(calc.add(2, 3)).toBe(5);
});
```

**En CI/CD:**

```yaml
unit-tests:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - run: npm ci
    - run: npm test -- --coverage
    - name: Check coverage threshold
      run: |
        coverage=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
        if (( $(echo "$coverage < 80" | bc -l) )); then
          echo "Coverage $coverage% is below 80%"
          exit 1
        fi
```

#### 2. Integration Tests (Pruebas de Integración)

**Características:**
- Prueban múltiples componentes juntos
- Pueden usar bases de datos de test
- Más lentas (segundos)
- Menos que unit tests (15% de tus tests)

**Ejemplo:**

```javascript
// api.integration.test.js
test('POST /users creates user and returns 201', async () => {
  const response = await request(app)
    .post('/users')
    .send({ name: 'John', email: 'john@example.com' });
  
  expect(response.status).toBe(201);
  expect(response.body).toHaveProperty('id');
  
  // Verificar en DB
  const user = await db.users.findOne({ email: 'john@example.com' });
  expect(user).toBeDefined();
});
```

**En CI/CD:**

```yaml
integration-tests:
  runs-on: ubuntu-latest
  services:
    postgres:
      image: postgres:15
      env:
        POSTGRES_PASSWORD: testpass
      options: >-
        --health-cmd pg_isready
        --health-interval 10s
        --health-timeout 5s
        --health-retries 5
  steps:
    - uses: actions/checkout@v3
    - run: npm ci
    - run: npm run test:integration
      env:
        DATABASE_URL: postgresql://postgres:testpass@localhost:5432/testdb
```

#### 3. End-to-End Tests (Pruebas E2E)

**Características:**
- Simulan usuario real
- Prueban flujo completo
- Muy lentas (minutos)
- Pocas pero críticas (5% de tus tests)

**Ejemplo con Playwright:**

```javascript
// e2e/login.spec.js
test('user can login', async ({ page }) => {
  await page.goto('https://myapp.com/login');
  
  await page.fill('input[name="email"]', 'user@test.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL('https://myapp.com/dashboard');
  await expect(page.locator('h1')).toContainText('Welcome');
});
```

**En CI/CD:**

```yaml
e2e-tests:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
    - run: npm ci
    - run: npx playwright install --with-deps
    
    - name: Start application
      run: npm start &
    
    - name: Wait for app
      run: npx wait-on http://localhost:3000
    
    - name: Run E2E tests
      run: npx playwright test
    
    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
```

#### 4. Performance Tests

```yaml
performance-tests:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Run load tests
      run: |
        docker run --rm -i grafana/k6 run - < loadtest.js
```

**Ejemplo loadtest.js:**

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 0 },   // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% requests < 500ms
  },
};

export default function () {
  const res = http.get('https://myapp.com/api/users');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  sleep(1);
}
```

#### 5. Security Tests

```yaml
security-scan:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    # Dependency vulnerability scanning
    - name: Run Snyk
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
    
    # SAST (Static Application Security Testing)
    - name: Run CodeQL
      uses: github/codeql-action/analyze@v2
    
    # Container scanning
    - name: Scan Docker image
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: myapp:latest
        severity: 'CRITICAL,HIGH'
```

### Test Strategy en Pipeline

```yaml
name: Test Suite

on: [push, pull_request]

jobs:
  # Stage 1: Fast tests (siempre se ejecutan)
  fast-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm run lint           # 30s
      - run: npm run test:unit      # 2min
  
  # Stage 2: Medium tests (después de fast tests)
  integration-tests:
    needs: fast-tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm run test:integration  # 5min
  
  # Stage 3: Slow tests (solo en main o con label)
  e2e-tests:
    needs: integration-tests
    if: github.ref == 'refs/heads/main' || contains(github.event.pull_request.labels.*.name, 'e2e')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npx playwright test     # 10min
```

---

## FASE 4: Package (Empaquetado)

### Descripción

Empaquetar la aplicación en un formato listo para distribuir.

### Estrategias de Empaquetado

#### 1. Docker Images (Recomendado)

**Ventajas:**
- Consistencia entre entornos
- Incluye todas las dependencias
- Portable y versionado

**Ejemplo Dockerfile:**

```dockerfile
# Multi-stage build para optimizar tamaño
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Production image
FROM node:18-alpine

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

EXPOSE 3000
USER node

CMD ["node", "dist/server.js"]
```

**En CI/CD:**

```yaml
package:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          myorg/myapp:${{ github.sha }}
          myorg/myapp:latest
        cache-from: type=registry,ref=myorg/myapp:buildcache
        cache-to: type=registry,ref=myorg/myapp:buildcache,mode=max
```

#### 2. Semantic Versioning

```yaml
versioning:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
    
    - name: Determine version
      id: version
      run: |
        # Usar GitVersion o similar
        VERSION=$(git describe --tags --abbrev=0 2>/dev/null || echo "0.1.0")
        NEXT_VERSION=$(echo $VERSION | awk -F. '{$NF = $NF + 1;} 1' | sed 's/ /./g')
        echo "version=$NEXT_VERSION" >> $GITHUB_OUTPUT
    
    - name: Tag release
      run: |
        git tag v${{ steps.version.outputs.version }}
        git push origin v${{ steps.version.outputs.version }}
```

#### 3. Artifact Repository

```yaml
# Publicar a npm
publish-npm:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
      with:
        node-version: '18.x'
        registry-url: 'https://registry.npmjs.org'
    - run: npm ci
    - run: npm publish
      env:
        NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

# Publicar a PyPI
publish-pypi:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-python@v4
    - run: pip install build twine
    - run: python -m build
    - run: twine upload dist/*
      env:
        TWINE_USERNAME: __token__
        TWINE_PASSWORD: ${{ secrets.PYPI_TOKEN }}
```

---

## FASE 5: Deploy (Despliegue)

### Descripción

Llevar la aplicación a un entorno donde pueda ser usado.

### Tipos de Despliegue

#### 1. Deploy to Staging (Pre-producción)

```yaml
deploy-staging:
  needs: [build, test, package]
  runs-on: ubuntu-latest
  environment:
    name: staging
    url: https://staging.myapp.com
  steps:
    - name: Deploy to staging
      run: |
        # Ejemplo con kubectl
        kubectl set image deployment/myapp \
          myapp=myorg/myapp:${{ github.sha }} \
          --namespace=staging
        
        kubectl rollout status deployment/myapp \
          --namespace=staging \
          --timeout=5m
```

#### 2. Deploy to Production

```yaml
deploy-production:
  needs: deploy-staging
  runs-on: ubuntu-latest
  environment:
    name: production
    url: https://myapp.com
  # Requiere aprobación manual
  steps:
    - name: Deploy to production
      run: |
        kubectl set image deployment/myapp \
          myapp=myorg/myapp:${{ github.sha }} \
          --namespace=production
        
        kubectl rollout status deployment/myapp \
          --namespace=production \
          --timeout=10m
    
    - name: Smoke test
      run: |
        response=$(curl -s -o /dev/null -w "%{http_code}" https://myapp.com/health)
        if [ $response -ne 200 ]; then
          echo "Health check failed"
          kubectl rollout undo deployment/myapp --namespace=production
          exit 1
        fi
```

#### 3. Ejemplos por Plataforma

**Heroku:**

```yaml
deploy-heroku:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - uses: akhileshns/heroku-deploy@v3.12.12
      with:
        heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
        heroku_app_name: "myapp"
        heroku_email: "myemail@example.com"
```

**Vercel:**

```yaml
deploy-vercel:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - uses: amondnet/vercel-action@v20
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-org-id: ${{ secrets.ORG_ID }}
        vercel-project-id: ${{ secrets.PROJECT_ID }}
        vercel-args: '--prod'
```

**AWS ECS:**

```yaml
deploy-ecs:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    
    - name: Deploy to ECS
      run: |
        aws ecs update-service \
          --cluster my-cluster \
          --service my-service \
          --force-new-deployment
```

---

## FASE 6: Monitor (Monitorización)

### Descripción

Verificar que el despliegue fue exitoso y la aplicación funciona correctamente.

### Post-Deployment Checks

```yaml
monitor:
  needs: deploy-production
  runs-on: ubuntu-latest
  steps:
    - name: Health check
      run: |
        for i in {1..5}; do
          if curl -f https://myapp.com/health; then
            echo "Health check passed"
            exit 0
          fi
          echo "Attempt $i failed, retrying..."
          sleep 10
        done
        exit 1
    
    - name: Smoke tests
      run: |
        # Test critical endpoints
        curl -f https://myapp.com/api/users || exit 1
        curl -f https://myapp.com/api/products || exit 1
    
    - name: Notify team
      if: failure()
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: 'Deployment failed! Rolling back...'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Métricas a Monitorizar

```yaml
1. Application metrics:
   - Response time
   - Error rate
   - Throughput (requests/second)

2. Infrastructure metrics:
   - CPU usage
   - Memory usage
   - Disk I/O

3. Business metrics:
   - Active users
   - Conversion rate
   - Revenue
```

---

## Pipeline Completo: Ejemplo Real

```yaml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ============================================
  # FASE CI: Source, Build, Test
  # ============================================
  
  lint:
    name: Code Quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check
  
  test-unit:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:unit -- --coverage
      - uses: codecov/codecov-action@v3
  
  test-integration:
    name: Integration Tests
    needs: test-unit
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: testpass
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/test
  
  build:
    name: Build Application
    needs: [lint, test-unit]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v3
        with:
          name: build
          path: dist/
  
  # ============================================
  # FASE CD: Package, Deploy
  # ============================================
  
  package:
    name: Build Docker Image
    needs: [build, test-integration]
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v3
      
      - name: Log in to registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
  
  deploy-staging:
    name: Deploy to Staging
    needs: package
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.myapp.com
    steps:
      - name: Deploy to staging
        run: |
          echo "Deploying to staging..."
          # kubectl apply -f k8s/staging/
      
      - name: Run smoke tests
        run: |
          sleep 30
          curl -f https://staging.myapp.com/health || exit 1
  
  deploy-production:
    name: Deploy to Production
    needs: package
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com
    steps:
      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          # kubectl apply -f k8s/production/
      
      - name: Health check
        run: |
          for i in {1..10}; do
            if curl -f https://myapp.com/health; then
              exit 0
            fi
            sleep 10
          done
          exit 1
      
      - name: Notify on failure
        if: failure()
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          text: 'Production deployment failed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Resumen de Fases

| Fase | Objetivo | Duración típica | Fallos aceptables |
|------|----------|-----------------|-------------------|
| **Source** | Obtener código | 5-10s | 0% |
| **Build** | Compilar/empaquetar | 1-5 min | 0% |
| **Test** | Verificar calidad | 2-10 min | 0% |
| **Package** | Crear artefacto | 1-3 min | 0% |
| **Deploy Staging** | Desplegar a pre-prod | 2-5 min | <5% |
| **Deploy Production** | Desplegar a prod | 2-5 min | <1% |
| **Monitor** | Verificar funcionamiento | Continuo | <1% |

**Total pipeline time ideal: 10-30 minutos**

---

## Próximos Pasos

En las siguientes secciones profundizaremos en:

- **Plataformas CI/CD**: GitHub Actions, GitLab CI, Jenkins en detalle
- **Automatización**: Triggers avanzados y workflows complejos
- **Secretos**: Gestión segura de credenciales
- **Estrategias de despliegue**: Blue-Green, Canary, Rolling

¡Continuemos construyendo pipelines robustos! 🚀
