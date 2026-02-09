# Automatización de Despliegues

## Introducción

La automatización de despliegues es el corazón de la Entrega Continua. Elimina intervención manual, reduce errores y acelera time-to-market.

---

## Triggers Automáticos

### 1. Push to Branch

```yaml
on:
  push:
    branches:
      - main          # Deploy production
      - develop       # Deploy staging
      - 'release/**'  # Deploy pre-production
```

### 2. Pull Request

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches: [main]
```

### 3. Tags (Releases)

```yaml
on:
  push:
    tags:
      - 'v*.*.*'  # v1.2.3

jobs:
  release:
    steps:
      - name: Get version
        run: echo "VERSION=${GITHUB_REF#refs/tags/}" >> $GITHUB_ENV
```

### 4. Schedule (Cron)

```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM diario
    - cron: '0 */6 * * *'  # Cada 6 horas
```

### 5. Manual (workflow_dispatch)

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        type: choice
        options:
          - staging
          - production
      version:
        description: 'Version to deploy'
        required: false
```

### 6. Workflow Call (Reutilizable)

```yaml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
```

---

## Estrategias de Deploy Automático

### Estrategia 1: Develop → Staging, Main → Production

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
            echo "environment=production" >> $GITHUB_OUTPUT
            echo "url=https://api.example.com" >> $GITHUB_OUTPUT
          else
            echo "environment=staging" >> $GITHUB_OUTPUT
            echo "url=https://staging.example.com" >> $GITHUB_OUTPUT
          fi
      
      - name: Deploy to ${{ steps.env.outputs.environment }}
        run: |
          echo "Deploying to ${{ steps.env.outputs.environment }}"
          ./deploy.sh ${{ steps.env.outputs.environment }}
```

### Estrategia 2: Feature Branch → Preview Environment

```yaml
name: Preview Deployments

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy-preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy preview
        run: |
          BRANCH_NAME="${{ github.head_ref }}"
          CLEAN_NAME=$(echo $BRANCH_NAME | tr '/' '-')
          
          # Deploy to preview-<branch-name>
          ./deploy-preview.sh $CLEAN_NAME
      
      - name: Comment PR
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '🚀 Preview deployed to https://preview-${{ github.head_ref }}.example.com'
            })
```

---

## Despliegue por Plataforma

### Heroku

```yaml
deploy-heroku:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - uses: akhileshns/heroku-deploy@v3.12.14
      with:
        heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
        heroku_app_name: "myapp-prod"
        heroku_email: "deploy@example.com"
        
    - name: Health check
      run: |
        sleep 30
        curl -f https://myapp-prod.herokuapp.com/health
```

### Vercel

```yaml
deploy-vercel:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to Vercel
      uses: amondnet/vercel-action@v20
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-org-id: ${{ secrets.ORG_ID }}
        vercel-project-id: ${{ secrets.PROJECT_ID }}
        vercel-args: '--prod'
```

### AWS ECS

```yaml
deploy-ecs:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    
    - name: Login to ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2
    
    - name: Build and push image
      run: |
        docker build -t ${{ steps.login-ecr.outputs.registry }}/myapp:${{ github.sha }} .
        docker push ${{ steps.login-ecr.outputs.registry }}/myapp:${{ github.sha }}
    
    - name: Update ECS service
      run: |
        aws ecs update-service \
          --cluster prod-cluster \
          --service myapp-service \
          --force-new-deployment
```

### Kubernetes

```yaml
deploy-k8s:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
    
    - name: Configure kubectl
      run: |
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig.yaml
        export KUBECONFIG=kubeconfig.yaml
    
    - name: Deploy
      run: |
        kubectl set image deployment/myapp \
          myapp=${{ secrets.REGISTRY }}/myapp:${{ github.sha }} \
          -n production
        
        kubectl rollout status deployment/myapp -n production --timeout=5m
```

### Docker Compose (servidor remoto)

```yaml
deploy-docker:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Copy files to server
      uses: appleboy/scp-action@master
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        source: "docker-compose.yml,.env"
        target: "/app"
    
    - name: Deploy with Docker Compose
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /app
          docker-compose pull
          docker-compose up -d
```

---

## Rollback Automático

### Rollback si Health Check Falla

```yaml
deploy-with-rollback:
  runs-on: ubuntu-latest
  steps:
    - name: Get current version
      id: current
      run: |
        CURRENT=$(kubectl get deployment myapp -n prod -o jsonpath='{.spec.template.spec.containers[0].image}')
        echo "image=$CURRENT" >> $GITHUB_OUTPUT
    
    - name: Deploy new version
      run: |
        kubectl set image deployment/myapp \
          myapp=myapp:${{ github.sha }} -n prod
        kubectl rollout status deployment/myapp -n prod
    
    - name: Health check
      id: health
      run: |
        for i in {1..10}; do
          if curl -f https://api.example.com/health; then
            echo "healthy=true" >> $GITHUB_OUTPUT
            exit 0
          fi
          sleep 10
        done
        echo "healthy=false" >> $GITHUB_OUTPUT
        exit 1
      continue-on-error: true
    
    - name: Rollback on failure
      if: steps.health.outputs.healthy == 'false'
      run: |
        echo "Health check failed, rolling back..."
        kubectl set image deployment/myapp \
          myapp=${{ steps.current.outputs.image }} -n prod
        kubectl rollout status deployment/myapp -n prod
        exit 1
```

---

## Notificaciones

### Slack

```yaml
- name: Notify Slack
  if: always()
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: |
      Deployment to ${{ env.ENVIRONMENT }}
      Result: ${{ job.status }}
      Commit: ${{ github.sha }}
      Author: ${{ github.actor }}
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
    fields: repo,message,commit,author,action,eventName,ref,workflow
```

### Discord

```yaml
- name: Discord notification
  uses: Ilshidur/action-discord@master
  env:
    DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK }}
  with:
    args: '🚀 Deployed {{ EVENT_PAYLOAD.repository.full_name }} to production'
```

### Email

```yaml
- name: Send email
  if: failure()
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 465
    username: ${{ secrets.EMAIL_USERNAME }}
    password: ${{ secrets.EMAIL_PASSWORD }}
    subject: Deployment Failed - ${{ github.repository }}
    to: team@example.com
    from: ci@example.com
    body: |
      Deployment failed for commit ${{ github.sha }}
      Check: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

---

## Deploy Scripts Reutilizables

### deploy.sh (genérico)

```bash
#!/bin/bash
set -e

ENVIRONMENT=$1
VERSION=${2:-latest}

echo "🚀 Deploying to $ENVIRONMENT (version: $VERSION)"

case $ENVIRONMENT in
  production)
    SERVER="prod.example.com"
    PORT=443
    ;;
  staging)
    SERVER="staging.example.com"
    PORT=443
    ;;
  *)
    echo "Unknown environment: $ENVIRONMENT"
    exit 1
    ;;
esac

# Build
echo "📦 Building..."
npm run build

# Deploy
echo "🔄 Deploying to $SERVER..."
rsync -avz --delete dist/ deploy@$SERVER:/var/www/app/

# Reload
echo "♻️  Reloading service..."
ssh deploy@$SERVER "sudo systemctl reload nginx"

# Health check
echo "🏥 Health check..."
sleep 5
if curl -f https://$SERVER/health; then
  echo "✅ Deployment successful!"
else
  echo "❌ Health check failed!"
  exit 1
fi
```

---

## Best Practices

1. **Idempotencia**: El mismo deploy múltiples veces = mismo resultado
2. **Atomic deploys**: All or nothing
3. **Zero downtime**: Blue-green o rolling updates
4. **Health checks**: Siempre verificar después de deploy
5. **Rollback automático**: Si algo falla
6. **Logging**: Registrar todo
7. **Notificaciones**: Mantener al equipo informado

---

**Próxima sección:** Gestión Segura de Secretos
