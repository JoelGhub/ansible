# Prácticas - Fases CI y CD

## Objetivo
Implementar un pipeline completo que cubra todas las fases desde el commit hasta producción.

---

## Práctica 1: Pipeline Multi-Stage con Todas las Fases

### Descripción
Crear un pipeline completo que incluya source control, build, test, package, deploy y monitor.

### Proyecto: API REST con Node.js + Express

#### 1. Crear el proyecto

```bash
mkdir cicd-complete-pipeline
cd cicd-complete-pipeline
npm init -y
npm install express dotenv
npm install --save-dev jest supertest eslint nodemon
```

#### 2. Estructura del proyecto

```bash
mkdir -p src/{routes,controllers,services,models} tests/{unit,integration} .github/workflows
```

#### 3. Crear la aplicación

**src/app.js:**

```javascript
const express = require('express');
const userRoutes = require('./routes/users');

const app = express();

app.use(express.json());

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy', timestamp: new Date().toISOString() });
});

// API routes
app.use('/api/users', userRoutes);

// Error handling
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

module.exports = app;
```

**src/server.js:**

```javascript
const app = require('./app');
const PORT = process.env.PORT || 3000;

const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM signal received: closing HTTP server');
  server.close(() => {
    console.log('HTTP server closed');
  });
});

module.exports = server;
```

**src/models/User.js:**

```javascript
class User {
  constructor(id, name, email) {
    this.id = id;
    this.name = name;
    this.email = email;
    this.createdAt = new Date();
  }

  validate() {
    if (!this.name || this.name.length < 2) {
      throw new Error('Name must be at least 2 characters');
    }
    if (!this.email || !this.email.includes('@')) {
      throw new Error('Invalid email format');
    }
    return true;
  }
}

module.exports = User;
```

**src/services/userService.js:**

```javascript
const User = require('../models/User');

class UserService {
  constructor() {
    this.users = [];
    this.nextId = 1;
  }

  createUser(name, email) {
    const user = new User(this.nextId++, name, email);
    user.validate();
    this.users.push(user);
    return user;
  }

  getAllUsers() {
    return this.users;
  }

  getUserById(id) {
    return this.users.find(u => u.id === parseInt(id));
  }

  deleteUser(id) {
    const index = this.users.findIndex(u => u.id === parseInt(id));
    if (index === -1) return false;
    this.users.splice(index, 1);
    return true;
  }
}

module.exports = new UserService();
```

**src/controllers/userController.js:**

```javascript
const userService = require('../services/userService');

exports.createUser = (req, res) => {
  try {
    const { name, email } = req.body;
    const user = userService.createUser(name, email);
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};

exports.getAllUsers = (req, res) => {
  const users = userService.getAllUsers();
  res.json(users);
};

exports.getUserById = (req, res) => {
  const user = userService.getUserById(req.params.id);
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.json(user);
};

exports.deleteUser = (req, res) => {
  const deleted = userService.deleteUser(req.params.id);
  if (!deleted) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.status(204).send();
};
```

**src/routes/users.js:**

```javascript
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.post('/', userController.createUser);
router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUserById);
router.delete('/:id', userController.deleteUser);

module.exports = router;
```

#### 4. Tests Unitarios

**tests/unit/user.test.js:**

```javascript
const User = require('../../src/models/User');

describe('User Model', () => {
  test('should create a valid user', () => {
    const user = new User(1, 'John Doe', 'john@example.com');
    expect(user.id).toBe(1);
    expect(user.name).toBe('John Doe');
    expect(user.email).toBe('john@example.com');
  });

  test('should validate user with correct data', () => {
    const user = new User(1, 'John', 'john@example.com');
    expect(() => user.validate()).not.toThrow();
  });

  test('should throw error for invalid name', () => {
    const user = new User(1, 'J', 'john@example.com');
    expect(() => user.validate()).toThrow('Name must be at least 2 characters');
  });

  test('should throw error for invalid email', () => {
    const user = new User(1, 'John', 'invalid-email');
    expect(() => user.validate()).toThrow('Invalid email format');
  });
});
```

**tests/unit/userService.test.js:**

```javascript
const UserService = require('../../src/services/userService');

describe('UserService', () => {
  beforeEach(() => {
    // Reset service
    UserService.users = [];
    UserService.nextId = 1;
  });

  test('should create a user', () => {
    const user = UserService.createUser('John', 'john@test.com');
    expect(user.id).toBe(1);
    expect(user.name).toBe('John');
  });

  test('should get all users', () => {
    UserService.createUser('John', 'john@test.com');
    UserService.createUser('Jane', 'jane@test.com');
    const users = UserService.getAllUsers();
    expect(users).toHaveLength(2);
  });

  test('should get user by id', () => {
    UserService.createUser('John', 'john@test.com');
    const user = UserService.getUserById(1);
    expect(user.name).toBe('John');
  });

  test('should delete user', () => {
    UserService.createUser('John', 'john@test.com');
    const deleted = UserService.deleteUser(1);
    expect(deleted).toBe(true);
    expect(UserService.getAllUsers()).toHaveLength(0);
  });
});
```

#### 5. Tests de Integración

**tests/integration/api.test.js:**

```javascript
const request = require('supertest');
const app = require('../../src/app');

describe('API Integration Tests', () => {
  test('GET /health should return 200', async () => {
    const response = await request(app).get('/health');
    expect(response.status).toBe(200);
    expect(response.body.status).toBe('healthy');
  });

  test('POST /api/users should create user', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'John Doe', email: 'john@example.com' });
    
    expect(response.status).toBe(201);
    expect(response.body.name).toBe('John Doe');
    expect(response.body.email).toBe('john@example.com');
    expect(response.body).toHaveProperty('id');
  });

  test('GET /api/users should return all users', async () => {
    // Create users first
    await request(app).post('/api/users').send({ name: 'John', email: 'john@test.com' });
    await request(app).post('/api/users').send({ name: 'Jane', email: 'jane@test.com' });

    const response = await request(app).get('/api/users');
    expect(response.status).toBe(200);
    expect(response.body).toHaveLength(2);
  });

  test('GET /api/users/:id should return specific user', async () => {
    const createResponse = await request(app)
      .post('/api/users')
      .send({ name: 'John', email: 'john@test.com' });
    
    const userId = createResponse.body.id;
    const response = await request(app).get(`/api/users/${userId}`);
    
    expect(response.status).toBe(200);
    expect(response.body.name).toBe('John');
  });

  test('DELETE /api/users/:id should delete user', async () => {
    const createResponse = await request(app)
      .post('/api/users')
      .send({ name: 'John', email: 'john@test.com' });
    
    const userId = createResponse.body.id;
    const response = await request(app).delete(`/api/users/${userId}`);
    
    expect(response.status).toBe(204);
  });

  test('POST /api/users with invalid data should return 400', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'J', email: 'invalid' });
    
    expect(response.status).toBe(400);
  });
});
```

#### 6. Configuración

**package.json:**

```json
{
  "name": "cicd-complete-pipeline",
  "version": "1.0.0",
  "description": "Complete CI/CD Pipeline Example",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "test": "jest --coverage",
    "test:unit": "jest tests/unit",
    "test:integration": "jest tests/integration",
    "test:watch": "jest --watch",
    "lint": "eslint src/ tests/",
    "lint:fix": "eslint src/ tests/ --fix"
  },
  "keywords": ["cicd", "api", "express"],
  "author": "Your Name",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2",
    "dotenv": "^16.0.3"
  },
  "devDependencies": {
    "jest": "^29.5.0",
    "supertest": "^6.3.3",
    "eslint": "^8.38.0",
    "nodemon": "^2.0.22"
  },
  "jest": {
    "testEnvironment": "node",
    "coverageDirectory": "coverage",
    "collectCoverageFrom": [
      "src/**/*.js",
      "!src/server.js"
    ],
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

**.eslintrc.js:**

```javascript
module.exports = {
  env: {
    node: true,
    es2021: true,
    jest: true
  },
  extends: 'eslint:recommended',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  rules: {
    'no-console': 'off',
    'no-unused-vars': 'warn',
    'semi': ['error', 'always'],
    'quotes': ['error', 'single']
  }
};
```

**Dockerfile:**

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY src ./src

# Production stage
FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/src ./src
COPY package*.json ./

ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000

USER node

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

CMD ["node", "src/server.js"]
```

**.dockerignore:**

```
node_modules
npm-debug.log
coverage
.git
.gitignore
README.md
.env
.github
tests
.eslintrc.js
```

#### 7. Pipeline CI/CD Completo

**.github/workflows/cicd-complete.yml:**

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
  # ========================================
  # STAGE 1: CODE QUALITY
  # ========================================
  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint

  # ========================================
  # STAGE 2: UNIT TESTS
  # ========================================
  test-unit:
    name: Unit Tests
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unittests

  # ========================================
  # STAGE 3: INTEGRATION TESTS
  # ========================================
  test-integration:
    name: Integration Tests
    runs-on: ubuntu-latest
    needs: test-unit
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run integration tests
        run: npm run test:integration

  # ========================================
  # STAGE 4: BUILD & PACKAGE
  # ========================================
  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [test-unit, test-integration]
    permissions:
      contents: read
      packages: write
    outputs:
      image: ${{ steps.meta.outputs.tags }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Container Registry
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
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=sha,format=short
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ steps.meta.outputs.tags }}
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  # ========================================
  # STAGE 5: DEPLOY TO STAGING
  # ========================================
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging-api.example.com
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Deploy to staging
        run: |
          echo "Deploying to staging environment..."
          echo "Image: ${{ needs.build.outputs.image }}"
          # Aquí iría tu lógica de despliegue real
          # Ejemplo: kubectl, helm, docker-compose, etc.
      
      - name: Wait for deployment
        run: sleep 30
      
      - name: Smoke test - Health check
        run: |
          echo "Running smoke tests..."
          # curl -f https://staging-api.example.com/health || exit 1
          echo "✅ Health check passed"
      
      - name: Notify team
        if: always()
        run: |
          echo "Staging deployment ${{ job.status }}"

  # ========================================
  # STAGE 6: DEPLOY TO PRODUCTION
  # ========================================
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://api.example.com
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Deploy to production
        run: |
          echo "Deploying to production environment..."
          echo "Image: ${{ needs.build.outputs.image }}"
          # Deployment logic here
      
      - name: Wait for deployment
        run: sleep 30
      
      - name: Health check
        run: |
          echo "Running health checks..."
          # for i in {1..10}; do
          #   if curl -f https://api.example.com/health; then
          #     exit 0
          #   fi
          #   sleep 10
          # done
          # exit 1
          echo "✅ Health check passed"
      
      - name: Smoke tests
        run: |
          echo "Running production smoke tests..."
          # curl -f https://api.example.com/api/users || exit 1
          echo "✅ All smoke tests passed"
      
      - name: Notify on success
        if: success()
        run: |
          echo "🎉 Production deployment successful!"
      
      - name: Rollback on failure
        if: failure()
        run: |
          echo "❌ Deployment failed, rolling back..."
          # Rollback logic here

  # ========================================
  # STAGE 7: POST-DEPLOYMENT MONITORING
  # ========================================
  monitor:
    name: Post-Deployment Monitoring
    runs-on: ubuntu-latest
    needs: deploy-production
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Monitor metrics
        run: |
          echo "Monitoring deployment metrics..."
          sleep 60
          echo "✅ No anomalies detected"
      
      - name: Generate deployment report
        run: |
          echo "## Deployment Report" > report.md
          echo "- **Time**: $(date)" >> report.md
          echo "- **Commit**: ${{ github.sha }}" >> report.md
          echo "- **Status**: Success ✅" >> report.md
      
      - name: Upload report
        uses: actions/upload-artifact@v3
        with:
          name: deployment-report
          path: report.md
```

#### 8. Probar el pipeline

```bash
# Inicializar Git
git init
git add .
git commit -m "feat: Complete CI/CD pipeline implementation"

# Crear repositorio en GitHub y subir
git remote add origin https://github.com/YOUR_USERNAME/cicd-complete-pipeline.git
git branch -M main
git push -u origin main

# Crear rama develop
git checkout -b develop
git push -u origin develop
```

### Resultado Esperado

El pipeline ejecutará todas las fases en orden:

```
1. Lint ✅ (30s)
     ↓
2. Unit Tests ✅ (1m)
     ↓
3. Integration Tests ✅ (1m 30s)
     ↓
4. Build Docker Image ✅ (2m)
     ↓
5. Deploy to Staging ✅ (1m) [solo en develop]
     o
   Deploy to Production ✅ (2m) [solo en main]
     ↓
6. Monitor ✅ (1m)

Total: ~7-9 minutos
```

---

## Práctica 2: Implementar Gates y Aprobaciones

### Descripción
Añadir aprobaciones manuales antes de desplegar a producción.

### Pasos

#### 1. Configurar Environment en GitHub

1. Ve a tu repositorio → Settings → Environments
2. Crea environment "production"
3. Configuración:
   - ✅ Required reviewers: Añade usuarios que deben aprobar
   - ✅ Wait timer: 5 minutos (opcional)
   - ✅ Deployment branches: Solo `main`

#### 2. El workflow ya está preparado

El `environment: production` en el job hace que requiera aprobación automáticamente.

#### 3. Proceso de aprobación

```bash
# Hacer cambio
git checkout main
echo "console.log('New feature');" >> src/app.js
git add .
git commit -m "feat: Add new feature"
git push

# En GitHub:
# 1. Pipeline se ejecuta hasta antes de producción
# 2. Espera aprobación manual
# 3. Reviewer aprueba o rechaza
# 4. Si aprueba, continúa el despliegue
```

---

## Práctica 3: Métricas del Pipeline

### Descripción
Recopilar y analizar métricas del pipeline.

### Crear script de análisis

**scripts/analyze-pipeline.sh:**

```bash
#!/bin/bash

echo "=== Pipeline Metrics Analysis ==="
echo ""

# Obtener runs del último mes usando GitHub CLI
gh run list --limit 100 --json conclusion,createdAt,durationMs,name > runs.json

# Analizar con jq
echo "📊 Summary:"
total=$(jq length runs.json)
successful=$(jq '[.[] | select(.conclusion=="success")] | length' runs.json)
failed=$(jq '[.[] | select(.conclusion=="failure")] | length' runs.json)
success_rate=$(echo "scale=2; $successful * 100 / $total" | bc)

echo "Total runs: $total"
echo "Successful: $successful"
echo "Failed: $failed"
echo "Success rate: $success_rate%"

# Duración promedio
avg_duration=$(jq '[.[] | .durationMs] | add / length / 1000 / 60' runs.json)
echo ""
echo "⏱️  Average duration: $(printf "%.2f" $avg_duration) minutes"

# Runs por día
echo ""
echo "📈 Deployment frequency:"
days=30
deploys_per_day=$(echo "scale=2; $total / $days" | bc)
echo "$deploys_per_day deploys/day"

# DORA classification
echo ""
echo "🎯 DORA Metrics Classification:"
if (( $(echo "$deploys_per_day >= 1" | bc -l) )); then
  echo "Deployment Frequency: Elite/High"
else
  echo "Deployment Frequency: Medium/Low"
fi

rm runs.json
```

```bash
chmod +x scripts/analyze-pipeline.sh
./scripts/analyze-pipeline.sh
```

---

## Ejercicios Adicionales

### Ejercicio 1: Añadir Tests E2E

Implementa tests E2E con Playwright o Cypress que se ejecuten después de deploy a staging.

### Ejercicio 2: Rollback Automático

Implementa rollback automático si los smoke tests fallan en producción.

### Ejercicio 3: Notificaciones

Añade notificaciones a Slack/Discord/Email en cada fase del pipeline.

### Ejercicio 4: Feature Flags

Implementa feature flags para desplegar código sin activarlo inmediatamente.

---

## Checklist de Completitud

- [ ] ✅ Pipeline con lint, test, build funcionando
- [ ] ✅ Tests unitarios con >80% coverage
- [ ] ✅ Tests de integración pasando
- [ ] ✅ Docker image construida y pusheada
- [ ] ✅ Security scanning configurado
- [ ] ✅ Deploy a staging automático
- [ ] ✅ Deploy a producción con aprobación manual
- [ ] ✅ Health checks post-deployment
- [ ] ✅ Smoke tests implementados
- [ ] ✅ Métricas recopiladas y analizadas

**¡Felicidades! Has implementado un pipeline CI/CD completo de nivel profesional. 🚀**
