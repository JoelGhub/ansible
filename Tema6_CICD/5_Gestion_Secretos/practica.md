# Prácticas - Gestión de Secretos

## Práctica 1: Configurar Secretos en GitHub

### Paso 1: Crear Aplicación con Secretos

**app.js:**
```javascript
require('dotenv').config();
const express = require('express');
const app = express();

// ❌ NUNCA así
// const apiKey = "sk_live_123";

// ✅ Desde variables de entorno
const apiKey = process.env.API_KEY;
const dbPassword = process.env.DB_PASSWORD;

app.get('/status', (req, res) => {
  res.json({
    status: 'ok',
    hasApiKey: !!apiKey,
    hasDbPassword: !!dbPassword
  });
});

app.listen(3000);
```

**.env.example:**
```bash
API_KEY=your-api-key-here
DB_PASSWORD=your-db-password
PORT=3000
```

**.gitignore:**
```
node_modules/
.env
.env.local
```

### Paso 2: Configurar GitHub Secrets

1. Ve a tu repositorio en GitHub
2. Settings → Secrets and variables → Actions
3. New repository secret
4. Añade:
   - Name: `API_KEY`, Value: `test-api-key-123`
   - Name: `DB_PASSWORD`, Value: `secure-password-456`

### Paso 3: Usar en Workflow

**.github/workflows/test-secrets.yml:**
```yaml
name: Test with Secrets

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
    
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - run: npm install
      
      - name: Run tests
        run: |
          echo "API_KEY is set: $([ -n "$API_KEY" ] && echo 'Yes' || echo 'No')"
          echo "DB_PASSWORD is set: $([ -n "$DB_PASSWORD" ] && echo 'Yes' || echo 'No')"
          npm test
```

---

## Práctica 2: Secrets por Entorno

### Setup Environments

1. Settings → Environments
2. Crear:
   - `staging` → Añadir `STAGING_API_KEY`
   - `production` → Añadir `PROD_API_KEY` (con required reviewers)

### Workflow Multi-Environment

```yaml
name: Multi-Environment Deploy

on:
  push:
    branches: [main, develop]

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to staging
        env:
          API_KEY: ${{ secrets.API_KEY }}  # Usa staging secret
        run: |
          echo "Deploying to staging"
          ./deploy.sh staging

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to production
        env:
          API_KEY: ${{ secrets.API_KEY }}  # Usa production secret
        run: |
          echo "Deploying to production"
          ./deploy.sh production
```

---

## Práctica 3: Scanning de Secretos

### Instalar git-secrets

```bash
# macOS
brew install git-secrets

# Linux
git clone https://github.com/awslabs/git-secrets.git
cd git-secrets
sudo make install

# Configurar en repo
cd your-repo
git secrets --install
git secrets --register-aws
```

### Pre-commit Hook

**.pre-commit-config.yaml:**
```yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
        exclude: package-lock.json
```

**Instalar:**
```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files
```

### CI Secret Scanning

```yaml
name: Security Scan

on: [push, pull_request]

jobs:
  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: TruffleHog Secret Scan
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
```

---

## Práctica 4: Docker Build Secrets

### Dockerfile con Build Secrets

```dockerfile
# syntax=docker/dockerfile:1.4

FROM node:18-alpine AS builder

WORKDIR /app

# Usar build secret para npm token
RUN --mount=type=secret,id=npm_token \
    if [ -f /run/secrets/npm_token ]; then \
      echo "//registry.npmjs.org/:_authToken=$(cat /run/secrets/npm_token)" > ~/.npmrc; \
    fi

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build && rm -f ~/.npmrc

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

CMD ["node", "dist/server.js"]
```

### Build con Secret

```bash
# Crear secret file
echo "your-npm-token" > .npm-token

# Build con secret
docker buildx build \
  --secret id=npm_token,src=.npm-token \
  -t myapp:latest .

# Limpiar
rm .npm-token
```

### En GitHub Actions

```yaml
- name: Build Docker image with secrets
  uses: docker/build-push-action@v4
  with:
    context: .
    push: true
    tags: myapp:latest
    secrets: |
      "npm_token=${{ secrets.NPM_TOKEN }}"
```

---

## Práctica 5: HashiCorp Vault (Opcional)

### Vault en Docker

```bash
# Iniciar Vault en modo dev
docker run --cap-add=IPC_LOCK \
  -e 'VAULT_DEV_ROOT_TOKEN_ID=myroot' \
  -p 8200:8200 \
  -d --name=vault vault

# Configurar cliente
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='myroot'

# Añadir secretos
vault kv put secret/myapp \
  api_key='secret-api-key-123' \
  db_password='secure-password'

# Leer secretos
vault kv get secret/myapp
```

### GitHub Actions con Vault

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Import secrets from Vault
        uses: hashicorp/vault-action@v2
        with:
          url: http://vault.example.com:8200
          token: ${{ secrets.VAULT_TOKEN }}
          secrets: |
            secret/data/myapp api_key | API_KEY ;
            secret/data/myapp db_password | DB_PASSWORD
      
      - name: Deploy
        run: |
          echo "Using API_KEY from Vault"
          ./deploy.sh
```

---

## Práctica 6: Rotate Secrets

### Script de Rotación

**rotate-api-key.sh:**
```bash
#!/bin/bash
set -e

# Generar nueva API key
NEW_KEY=$(openssl rand -hex 32)

# Actualizar en servicio externo
curl -X POST https://api.example.com/keys/rotate \
  -H "Authorization: Bearer $OLD_KEY" \
  -d "new_key=$NEW_KEY"

# Actualizar en GitHub via API
gh secret set API_KEY --body "$NEW_KEY"

echo "✅ API key rotated successfully"
```

### Automated Rotation Workflow

```yaml
name: Rotate Secrets

on:
  schedule:
    - cron: '0 0 1 * *'  # 1º de cada mes
  workflow_dispatch:

jobs:
  rotate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Rotate API keys
        env:
          OLD_KEY: ${{ secrets.API_KEY }}
          GH_TOKEN: ${{ secrets.PAT_TOKEN }}
        run: |
          chmod +x rotate-api-key.sh
          ./rotate-api-key.sh
      
      - name: Notify team
        run: echo "🔄 Secrets rotated successfully"
```

---

## Ejercicios

### Ejercicio 1: Multi-Secret App
Crea una app que use 5+ secretos diferentes y despliégala.

### Ejercicio 2: Secret Audit
Implementa logging de quién/cuándo se usan secretos.

### Ejercicio 3: Vault Integration
Integra HashiCorp Vault en tu pipeline CI/CD.

### Ejercicio 4: Emergency Rotation
Crea un workflow manual para rotar todos los secretos en emergencia.

---

## Checklist

- [ ] Secretos configurados en GitHub
- [ ] Environment-based secrets
- [ ] Pre-commit hook para detección
- [ ] CI scanning implementado
- [ ] Docker build secrets configurados
- [ ] .env en .gitignore
- [ ] Secrets rotation script creado
- [ ] Audit logging implementado

**¡Secretos gestionados de forma segura! 🔒**
