# Ejercicio Final Sencillo - CI/CD Básico

## Objetivo

Implementar un **pipeline CI/CD básico** para familiarizarte con los conceptos fundamentales.

**Tiempo estimado:** 2-3 horas  
**Dificultad:** Baja

---

## Proyecto: Blog API Simple

Crear una API REST minimalista para un blog con Node.js + Express.

---

## Requisitos

### Funcionalidad
- [ ] GET /posts - Listar posts
- [ ] GET /posts/:id - Ver post
- [ ] POST /posts - Crear post
- [ ] GET /health - Health check

### Pipeline CI
- [ ] Tests automáticos
- [ ] Linting
- [ ] Build Docker image

### Pipeline CD
- [ ] Deploy automático a Heroku/Vercel
- [ ] Health check post-deploy

---

## Paso a Paso

### 1. Crear la Aplicación (15 min)

```bash
mkdir blog-api-cicd
cd blog-api-cicd
npm init -y
npm install express
npm install --save-dev jest supertest eslint
```

**src/app.js:**
```javascript
const express = require('express');
const app = express();

app.use(express.json());

let posts = [
  { id: 1, title: 'First Post', content: 'Hello World' }
];

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.get('/posts', (req, res) => {
  res.json(posts);
});

app.get('/posts/:id', (req, res) => {
  const post = posts.find(p => p.id === parseInt(req.params.id));
  if (!post) return res.status(404).json({ error: 'Not found' });
  res.json(post);
});

app.post('/posts', (req, res) => {
  const post = {
    id: posts.length + 1,
    title: req.body.title,
    content: req.body.content
  };
  posts.push(post);
  res.status(201).json(post);
});

module.exports = app;
```

**src/server.js:**
```javascript
const app = require('./app');
const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### 2. Tests (15 min)

**tests/app.test.js:**
```javascript
const request = require('supertest');
const app = require('../src/app');

describe('Blog API', () => {
  test('GET /health returns 200', async () => {
    const res = await request(app).get('/health');
    expect(res.status).toBe(200);
    expect(res.body.status).toBe('healthy');
  });

  test('GET /posts returns array', async () => {
    const res = await request(app).get('/posts');
    expect(res.status).toBe(200);
    expect(Array.isArray(res.body)).toBe(true);
  });

  test('POST /posts creates post', async () => {
    const res = await request(app)
      .post('/posts')
      .send({ title: 'Test', content: 'Content' });
    expect(res.status).toBe(201);
    expect(res.body).toHaveProperty('id');
  });
});
```

**package.json:**
```json
{
  "scripts": {
    "start": "node src/server.js",
    "test": "jest",
    "lint": "eslint src/ tests/"
  }
}
```

### 3. Pipeline CI (20 min)

**.github/workflows/ci.yml:**
```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run lint
      - run: npm test
```

### 4. Deploy a Heroku (30 min)

**Procfile:**
```
web: node src/server.js
```

**Pipeline CD:**

**.github/workflows/deploy.yml:**
```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: akhileshns/heroku-deploy@v3.12.14
        with:
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          heroku_app_name: "tu-app-name"
          heroku_email: "tu-email@example.com"
      
      - name: Health check
        run: |
          sleep 30
          curl -f https://tu-app-name.herokuapp.com/health
```

### 5. Smoke Tests (15 min)

**smoke-tests.sh:**
```bash
#!/bin/bash
URL=$1

echo "🔥 Running smoke tests..."

# Health check
curl -f $URL/health || exit 1
echo "✅ Health OK"

# Posts endpoint
curl -f $URL/posts || exit 1
echo "✅ Posts OK"

echo "✅ All tests passed!"
```

Añadir al workflow:
```yaml
- name: Smoke tests
  run: |
    chmod +x smoke-tests.sh
    ./smoke-tests.sh https://tu-app-name.herokuapp.com
```

---

## Testing Local

```bash
# Tests
npm test

# Lint
npm run lint

# Run locally
npm start

# Test endpoints
curl http://localhost:3000/health
curl http://localhost:3000/posts
```

---

## Subir a GitHub

```bash
git init
git add .
git commit -m "Initial commit: Blog API with CI/CD"
git remote add origin https://github.com/tu-usuario/blog-api-cicd.git
git push -u origin main
```

---

## Verificar Pipeline

1. Ve a GitHub → Actions
2. Verás el workflow ejecutándose
3. Espera a que termine (2-3 min)
4. Si es verde ✅, el pipeline funciona!

---

## Checklist de Completitud

- [ ] ✅ App funcionando localmente
- [ ] ✅ Tests pasando
- [ ] ✅ Linting OK
- [ ] ✅ Pipeline CI ejecutándose en GitHub
- [ ] ✅ Deploy automático funcionando
- [ ] ✅ Health check post-deploy pasando
- [ ] ✅ Smoke tests implementados

---

## Extensiones Opcionales (+30 min)

### Añadir Badge al README

```markdown
# Blog API

![CI](https://github.com/tu-usuario/blog-api-cicd/workflows/CI/badge.svg)

Simple blog API with CI/CD pipeline.
```

### Añadir Docker

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY src ./src
EXPOSE 3000
CMD ["node", "src/server.js"]
```

### Añadir Environments

1. Settings → Environments
2. Crear "production"
3. Añadir required reviewers

---

## Resultado Final

Al completar este ejercicio tendrás:

✅ Pipeline CI que ejecuta tests automáticamente  
✅ Deploy automático a producción en cada push  
✅ Health checks post-despliegue  
✅ Experiencia práctica con GitHub Actions  

**¡Felicidades! Has completado tu primer pipeline CI/CD. 🎉**

---

## Próximo Paso

Una vez domines este ejercicio básico, intenta el [Ejercicio Final Completo](practica.md) que incluye:
- Tests avanzados
- Múltiples entornos
- Estrategias de despliegue
- Monitorización
- Y mucho más!
