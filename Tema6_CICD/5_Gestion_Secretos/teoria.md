# Gestión de Secretos

## ¿Por Qué Son Importantes los Secretos?

Los **secretos** son información sensible que no debe estar en el código:
- API keys
- Contraseñas de bases de datos
- Tokens de autenticación
- Certificados SSL
- Credenciales cloud (AWS, Azure, GCP)

**❌ NUNCA hagas esto:**
```javascript
const API_KEY = "sk_live_123456789";  // ❌ Expuesto en código
const DB_PASSWORD = "mypassword123";   // ❌ En repositorio
```

---

## Gestión de Secretos por Plataforma

### GitHub Actions Secrets

#### Crear Secretos

1. Repositorio → Settings → Secrets and variables → Actions
2. New repository secret
3. Name: `API_KEY`, Value: `tu-api-key`

#### Tipos de Secretos

```yaml
# Repository secrets (un repo)
${{ secrets.API_KEY }}

# Organization secrets (toda la org)
${{ secrets.ORG_WIDE_TOKEN }}

# Environment secrets (por environment)
environment: production
# Puede usar secrets.PROD_API_KEY
```

#### Uso en Workflows

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      # Variables de entorno desde secrets
      API_KEY: ${{ secrets.API_KEY }}
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
    
    steps:
      - name: Use secrets
        run: |
          echo "API_KEY is set"
          # El valor nunca se muestra en logs
          curl -H "Authorization: Bearer $API_KEY" https://api.example.com
      
      - name: Pass to action
        uses: some/action@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}  # Token automático
```

**Secretos automáticos:**
- `GITHUB_TOKEN`: Token temporal para API de GitHub

---

### GitLab CI/CD Variables

#### Crear Variables

1. Project → Settings → CI/CD → Variables → Expand
2. Add variable
3. Key: `API_KEY`, Value: `tu-valor`
4. Opciones:
   - ✅ Protect: Solo ramas protegidas
   - ✅ Mask: Ocultar en logs
   - File: Variable como archivo

#### Uso

```yaml
deploy:
  script:
    - echo "Using $API_KEY"
    - deploy --token $API_KEY
  variables:
    CUSTOM_VAR: "value"
```

---

### Jenkins Credentials

#### Crear Credentials

1. Manage Jenkins → Credentials
2. Add Credentials
3. Tipo: Secret text, Username/password, SSH key, etc.

#### Uso en Jenkinsfile

```groovy
pipeline {
    environment {
        API_KEY = credentials('api-key-id')
        AWS_CREDS = credentials('aws-credentials')
    }
    
    stages {
        stage('Deploy') {
            steps {
                sh '''
                    echo "Using API key"
                    deploy --api-key $API_KEY
                '''
            }
        }
    }
}
```

---

## Gestores de Secretos Externos

### HashiCorp Vault

**Características:**
- Almacenamiento centralizado
- Secrets dinámicos
- Auditoría completa
- Rotación automática

**Ejemplo con GitHub Actions:**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Import Secrets from Vault
        uses: hashicorp/vault-action@v2
        with:
          url: https://vault.example.com
          token: ${{ secrets.VAULT_TOKEN }}
          secrets: |
            secret/data/production db_password | DB_PASSWORD ;
            secret/data/production api_key | API_KEY
      
      - name: Use secrets
        run: |
          echo "DB_PASSWORD is available"
          ./deploy.sh
```

### AWS Secrets Manager

```yaml
- name: Get secrets from AWS
  run: |
    SECRET=$(aws secretsmanager get-secret-value \
      --secret-id prod/api-key \
      --query SecretString \
      --output text)
    echo "::add-mask::$SECRET"
    echo "API_KEY=$SECRET" >> $GITHUB_ENV
```

### Azure Key Vault

```yaml
- uses: Azure/get-keyvault-secrets@v1
  with:
    keyvault: "myKeyVault"
    secrets: 'API-KEY, DB-PASSWORD'
  id: keyvault
  
- name: Use secrets
  run: echo "API Key: ${{ steps.keyvault.outputs.API-KEY }}"
```

---

## Buenas Prácticas

### 1. Nunca Hardcodear Secretos

```javascript
❌ const password = "mypassword123";
✅ const password = process.env.DB_PASSWORD;
```

### 2. Usar .env Files (Solo Local)

```bash
# .env (no committed)
API_KEY=dev-key-123
DB_PASSWORD=local-password
```

**.gitignore:**
```
.env
.env.local
.env.*.local
secrets/
```

### 3. Rotar Secretos Regularmente

```yaml
# Automated secret rotation
on:
  schedule:
    - cron: '0 0 1 * *'  # 1º de cada mes

jobs:
  rotate-secrets:
    runs-on: ubuntu-latest
    steps:
      - name: Rotate API keys
        run: ./rotate-keys.sh
```

### 4. Principio del Mínimo Privilegio

```yaml
# ❌ NO uses secrets con demasiado acceso
secrets.ADMIN_KEY

# ✅ Crea secrets específicos
secrets.DEPLOY_KEY  # Solo deploy
secrets.READ_ONLY_TOKEN  # Solo lectura
```

### 5. Auditar Acceso

```yaml
- name: Log secret usage
  run: |
    echo "Secret API_KEY used by ${{ github.actor }}" >> audit.log
```

### 6. Scanning de Secretos

**git-secrets:**
```bash
# Instalar
brew install git-secrets

# Configurar
git secrets --install
git secrets --register-aws

# Previene commits con secretos
```

**TruffleHog:**
```yaml
- name: Scan for secrets
  uses: trufflesecurity/trufflehog@main
  with:
    path: ./
    base: main
    head: HEAD
```

### 7. Masking en Logs

```yaml
- name: Add mask
  run: |
    SECRET_VALUE="my-secret-123"
    echo "::add-mask::$SECRET_VALUE"
    echo "Secret is: $SECRET_VALUE"  # Se mostrará como ***
```

---

## Secretos para Diferentes Entornos

### Approach 1: Environment-based

```yaml
deploy-staging:
  environment: staging
  steps:
    - run: deploy
      env:
        API_KEY: ${{ secrets.STAGING_API_KEY }}

deploy-production:
  environment: production
  steps:
    - run: deploy
      env:
        API_KEY: ${{ secrets.PROD_API_KEY }}
```

### Approach 2: Branch-based

```yaml
- name: Set environment
  run: |
    if [ "${{ github.ref }}" == "refs/heads/main" ]; then
      echo "API_KEY=${{ secrets.PROD_API_KEY }}" >> $GITHUB_ENV
    else
      echo "API_KEY=${{ secrets.STAGING_API_KEY }}" >> $GITHUB_ENV
    fi
```

---

## Secretos en Docker

### Dockerfile (Build Secrets)

```dockerfile
# docker buildx build --secret id=npm_token .

FROM node:18

RUN --mount=type=secret,id=npm_token \
    echo "//registry.npmjs.org/:_authToken=$(cat /run/secrets/npm_token)" > ~/.npmrc && \
    npm install && \
    rm ~/.npmrc
```

### Runtime Secrets

```yaml
services:
  app:
    image: myapp
    environment:
      - API_KEY=${API_KEY}
    secrets:
      - db_password

secrets:
  db_password:
    external: true
```

---

## Detección de Secretos Expuestos

### GitHub Secret Scanning

Automático en repos públicos. Detecta:
- AWS keys
- Azure tokens
- GitHub PATs
- Stripe keys
- Google API keys
- +100 patterns

### Pre-commit Hooks

**pre-commit-config.yaml:**
```yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

---

## Errores Comunes

### ❌ Error 1: Secreto en Logs

```yaml
- run: echo "API key is ${{ secrets.API_KEY }}"  # ❌ Expuesto
```

✅ **Solución:**
```yaml
- run: echo "API key is set"  # ✅ No mostrar valor
```

### ❌ Error 2: Secreto en Código

```javascript
// ❌ Committed to git
const apiKey = "sk_live_abc123";
```

✅ **Solución:**
```javascript
// ✅ Desde variable de entorno
const apiKey = process.env.API_KEY;
```

### ❌ Error 3: .env en Git

```bash
git add .env  # ❌ NO!
```

✅ **Solución:**
```bash
# .gitignore
.env
.env.local
.env.*.local
```

---

## Checklist de Seguridad

- [ ] ✅ Secretos en variables de entorno, no en código
- [ ] ✅ .env en .gitignore
- [ ] ✅ Secrets configurados en plataforma CI/CD
- [ ] ✅ Scanning de secretos habilitado
- [ ] ✅ Pre-commit hooks configurados
- [ ] ✅ Rotación de secretos planificada
- [ ] ✅ Acceso de secretos auditado
- [ ] ✅ Mínimo privilegio aplicado
- [ ] ✅ Secrets masked en logs

---

**Próxima sección:** Pruebas Post-Despliegue
