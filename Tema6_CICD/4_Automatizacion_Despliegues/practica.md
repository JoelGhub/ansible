# Prácticas - Automatización de Despliegues

## Práctica 1: Deploy Automático tras Push

### Aplicación Simple - Express API

**Crear proyecto:**
```bash
mkdir auto-deploy-demo && cd auto-deploy-demo
npm init -y
npm install express
```

**server.js:**
```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', version: process.env.VERSION || '1.0.0' });
});

app.get('/', (req, res) => {
  res.json({ message: 'Auto-deploy demo', version: process.env.VERSION });
});

app.listen(PORT, () => console.log(`Server on port ${PORT}`));
```

**Pipeline con auto-deploy:**

**.github/workflows/auto-deploy.yml:**
```yaml
name: Auto Deploy

on:
  push:
    branches: [main, develop]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Determine environment
        id: env
        run: |
          if [ "${{ github.ref }}" == "refs/heads/main" ]; then
            echo "env=production" >> $GITHUB_OUTPUT
          else
            echo "env=staging" >> $GITHUB_OUTPUT
          fi
      
      - name: Deploy to ${{ steps.env.outputs.env }}
        run: |
          echo "🚀 Deploying to ${{ steps.env.outputs.env }}"
          echo "Commit: ${{ github.sha }}"
          # Aquí tu lógica de deploy real
      
      - name: Notify
        run: echo "✅ Deployed to ${{ steps.env.outputs.env }}"
```

---

## Práctica 2: Deploy con Vercel (Gratis)

**Instalar Vercel CLI:**
```bash
npm i -g vercel
vercel login
```

**Obtener tokens:**
```bash
vercel whoami  # Tu username
cd my-project
vercel link    # Crear proyecto
# Guarda: ORG_ID y PROJECT_ID de .vercel/project.json
```

**GitHub Actions:**
```yaml
name: Vercel Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: '--prod'
      
      - name: Comment deployment URL
        run: echo "✅ Deployed to production"
```

**Secrets en GitHub:**
1. Settings → Secrets → New repository secret
2. Añadir: `VERCEL_TOKEN`, `ORG_ID`, `PROJECT_ID`

---

## Práctica 3: Preview Deployments en PR

**Netlify PR Previews:**

**.github/workflows/preview.yml:**
```yaml
name: Deploy Preview

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy-preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install
        run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v2
        with:
          publish-dir: './dist'
          production-deploy: false
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from PR #${{ github.event.number }}"
          enable-pull-request-comment: true
          enable-commit-comment: true
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

---

## Práctica 4: Deploy a VPS con SSH

**Setup:**
```bash
# Generar SSH key
ssh-keygen -t ed25519 -C "github-actions" -f deploy_key

# Copiar al servidor
ssh-copy-id -i deploy_key.pub user@server.com
```

**GitHub Actions:**
```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/myapp
            git pull origin main
            npm install
            npm run build
            pm2 reload myapp
```

**Secrets:**
- `SERVER_HOST`: IP o dominio
- `SERVER_USER`: usuario SSH
- `SSH_PRIVATE_KEY`: contenido de deploy_key (privada)

---

## Práctica 5: Rollback Automático

```yaml
name: Deploy with Auto-Rollback

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Save current deployment
        id: save
        run: |
          # Guardar versión actual para rollback
          echo "previous=v1.2.3" >> $GITHUB_OUTPUT
      
      - name: Deploy new version
        id: deploy
        run: |
          echo "Deploying version ${{ github.sha }}"
          ./deploy.sh
        continue-on-error: true
      
      - name: Health check
        id: health
        if: steps.deploy.outcome == 'success'
        run: |
          sleep 30
          for i in {1..5}; do
            if curl -f https://myapp.com/health; then
              echo "status=healthy" >> $GITHUB_OUTPUT
              exit 0
            fi
            sleep 10
          done
          echo "status=unhealthy" >> $GITHUB_OUTPUT
          exit 1
        continue-on-error: true
      
      - name: Rollback if unhealthy
        if: steps.health.outputs.status == 'unhealthy' || steps.deploy.outcome == 'failure'
        run: |
          echo "❌ Deployment failed or unhealthy"
          echo "🔄 Rolling back to ${{ steps.save.outputs.previous }}"
          ./deploy.sh ${{ steps.save.outputs.previous }}
          exit 1
      
      - name: Success notification
        if: steps.health.outputs.status == 'healthy'
        run: echo "✅ Deployment successful and healthy"
```

---

## Práctica 6: Scheduled Deployments

**Deploys nocturnos:**

```yaml
name: Nightly Deploy

on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM UTC
  workflow_dispatch:  # También manual

jobs:
  deploy-nightly:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy nightly build
        run: |
          VERSION="nightly-$(date +%Y%m%d)"
          echo "Deploying $VERSION"
          ./deploy.sh staging $VERSION
```

---

## Práctica 7: Multi-Environment Deploy

```yaml
name: Multi-Environment Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
          - dev
          - staging
          - production
      version:
        description: 'Version to deploy'
        required: false
        default: 'latest'

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment }}
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to ${{ github.event.inputs.environment }}
        run: |
          echo "Deploying to ${{ github.event.inputs.environment }}"
          echo "Version: ${{ github.event.inputs.version }}"
          ./deploy.sh ${{ github.event.inputs.environment }} ${{ github.event.inputs.version }}
```

**Uso:**
- Ve a Actions → Multi-Environment Deploy → Run workflow
- Selecciona environment y version
- Click "Run workflow"

---

## Ejercicios

### Ejercicio 1: Deploy con Docker
Automatiza build y push de imagen Docker, luego deploy a un servidor.

### Ejercicio 2: Canary Deployment
Despliega 10% del tráfico a nueva versión, si OK → 100%.

### Ejercicio 3: Cleanup Automático
Elimina preview deployments cuando se cierra el PR.

### Ejercicio 4: Deploy Approval
Requiere aprobación manual para production.

---

## Checklist

- [ ] Auto-deploy en push a main
- [ ] Preview deployments en PRs
- [ ] Deploy a plataforma cloud (Vercel/Netlify/Heroku)
- [ ] Deploy a VPS vía SSH
- [ ] Health checks post-deploy
- [ ] Rollback automático implementado
- [ ] Notificaciones configuradas
- [ ] Scheduled deploys (opcional)
- [ ] Multi-environment setup

**¡Despliegues automáticos dominados! 🚀**
