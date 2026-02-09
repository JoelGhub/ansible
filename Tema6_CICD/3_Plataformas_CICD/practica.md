# Prácticas - Plataformas CI/CD

## Práctica 1: Implementar el Mismo Pipeline en 3 Plataformas

### Objetivo
Crear un pipeline idéntico en GitHub Actions, GitLab CI y CircleCI.

### Aplicación de Ejemplo

**app.js:**
```javascript
function sum(a, b) { return a + b; }
function multiply(a, b) { return a * b; }
module.exports = { sum, multiply };
```

**app.test.js:**
```javascript
const { sum, multiply } = require('./app');
test('sum', () => expect(sum(2, 3)).toBe(5));
test('multiply', () => expect(multiply(2, 3)).toBe(6));
```

**package.json:**
```json
{
  "scripts": { "test": "jest" },
  "devDependencies": { "jest": "^29.0.0" }
}
```

---

### GitHub Actions

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
      - run: npm ci
      - run: npm test
```

---

### GitLab CI

**.gitlab-ci.yml:**
```yaml
image: node:18

stages:
  - test

test:
  stage: test
  script:
    - npm ci
    - npm test
```

---

### CircleCI

**.circleci/config.yml:**
```yaml
version: 2.1
jobs:
  test:
    docker:
      - image: cimg/node:18.0
    steps:
      - checkout
      - run: npm ci
      - run: npm test
workflows:
  test:
    jobs:
      - test
```

---

## Práctica 2: GitHub Actions Avanzado

### Multi-plataforma Matrix Build

```yaml
name: Multi-Platform CI

on: [push]

jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [16, 18, 20]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm test
```

### Reusable Workflows

**.github/workflows/reusable-build.yml:**
```yaml
name: Reusable Build

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
    outputs:
      artifact-id:
        value: ${{ jobs.build.outputs.artifact-id }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact-id: ${{ steps.upload.outputs.artifact-id }}
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci && npm run build
      - id: upload
        uses: actions/upload-artifact@v3
        with:
          name: build-${{ inputs.node-version }}
          path: dist/
```

**Uso:**
```yaml
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '18'
```

---

## Práctica 3: GitLab CI Avanzado

### Pipelines con DAG

```yaml
stages:
  - build
  - test
  - deploy

build:app:
  stage: build
  script: npm run build

test:unit:
  stage: test
  needs: [build:app]
  script: npm run test:unit

test:e2e:
  stage: test
  needs: [build:app]
  script: npm run test:e2e

deploy:
  stage: deploy
  needs: [test:unit, test:e2e]
  script: ./deploy.sh
```

### Templates Reutilizables

**.gitlab-ci-templates/node-test.yml:**
```yaml
.node_test:
  image: node:${NODE_VERSION}
  before_script:
    - npm ci
  script:
    - npm test
```

**Uso:**
```yaml
include:
  - '.gitlab-ci-templates/node-test.yml'

test:16:
  extends: .node_test
  variables:
    NODE_VERSION: "16"

test:18:
  extends: .node_test
  variables:
    NODE_VERSION: "18"
```

---

## Práctica 4: Jenkins Pipeline

### Setup Jenkins con Docker

```bash
docker run -d -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --name jenkins jenkins/jenkins:lts
```

### Jenkinsfile Completo

```groovy
pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS-18'
    }
    
    environment {
        CI = 'true'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
                archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh './deploy.sh'
            }
        }
    }
    
    post {
        always {
            junit 'test-results/*.xml'
            publishHTML([
                reportDir: 'coverage',
                reportFiles: 'index.html',
                reportName: 'Coverage Report'
            ])
        }
        success {
            slackSend color: 'good', message: "Build successful: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: 'danger', message: "Build failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
```

---

## Práctica 5: Migración de Pipelines

### Herramienta: gh-actions-converter

```bash
# Instalar
npm install -g gh-actions-importer

# Convertir Jenkinsfile a GitHub Actions
gh actions-importer convert jenkins Jenkinsfile

# Resultado: .github/workflows/jenkins-converted.yml
```

### Comparación Manual

**Jenkins:**
```groovy
stage('Test') {
    steps {
        sh 'npm test'
    }
}
```

**GitHub Actions:**
```yaml
- name: Test
  run: npm test
```

**GitLab CI:**
```yaml
test:
  script:
    - npm test
```

---

## Práctica 6: Self-Hosted Runners

### GitHub Actions Self-Hosted

```bash
# En tu servidor
mkdir actions-runner && cd actions-runner

# Descargar runner
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz

# Extraer
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz

# Configurar (usa el token de GitHub Settings → Actions → Runners)
./config.sh --url https://github.com/YOUR/REPO --token YOUR_TOKEN

# Ejecutar
./run.sh
```

**Uso en workflow:**
```yaml
jobs:
  build:
    runs-on: self-hosted  # 👈 Usa tu runner
    steps:
      - uses: actions/checkout@v3
      - run: npm test
```

---

## Ejercicios Adicionales

### Ejercicio 1: Monorepo Pipeline
Crea un pipeline que solo ejecute tests de los paquetes modificados en un monorepo.

### Ejercicio 2: Cross-Platform Build
Compila una aplicación Electron para Windows, macOS y Linux.

### Ejercicio 3: Dynamic Pipelines
GitLab: Genera stages dinámicamente basado en archivos modificados.

### Ejercicio 4: Jenkins Shared Library
Crea una shared library para reutilizar pasos comunes.

---

## Checklist

- [ ] Pipeline básico en GitHub Actions
- [ ] Pipeline básico en GitLab CI
- [ ] Pipeline básico en CircleCI (opcional)
- [ ] Matrix builds configurado
- [ ] Reusable workflows/templates
- [ ] Pipeline con aprobaciones manuales
- [ ] Self-hosted runner configurado (opcional)
- [ ] Migración entre plataformas practicada

**¡Ahora dominas múltiples plataformas CI/CD! 🎉**
