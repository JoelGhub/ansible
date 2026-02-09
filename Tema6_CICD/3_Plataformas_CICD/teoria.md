# Plataformas CI/CD

## Introducción

Existen múltiples plataformas para implementar pipelines CI/CD, cada una con sus ventajas y casos de uso específicos.

## Comparativa de Plataformas

| Plataforma | Tipo | Precio | Ideal para | Curva aprendizaje |
|------------|------|--------|------------|-------------------|
| **GitHub Actions** | Cloud | Free (limits) | Proyectos GitHub | Baja |
| **GitLab CI/CD** | Cloud/Self-hosted | Free tier | All-in-one DevOps | Media |
| **Jenkins** | Self-hosted | Free | Enterprise, legacy | Alta |
| **CircleCI** | Cloud | Free tier | Startups | Baja |
| **Travis CI** | Cloud | Free (OSS) | Open Source | Baja |
| **Azure DevOps** | Cloud | Free tier | Microsoft stack | Media |
| **AWS CodePipeline** | Cloud | Pay-per-use | AWS infrastructure | Media |

---

## GitHub Actions

### Características

- ✅ Integrado nativamente con GitHub
- ✅ Gran marketplace de actions reutilizables
- ✅ 2000 minutos gratis/mes (repos públicos ilimitados)
- ✅ Runners en Linux, Windows, macOS
- ✅ Self-hosted runners
- ✅ Matrix builds
- ✅ Environments con protecciones

### Estructura Básica

```yaml
name: Workflow Name
on: [push, pull_request]
jobs:
  job-name:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Step name
        run: echo "Hello World"
```

### Ejemplo Completo

```yaml
name: Node.js CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:

env:
  NODE_VERSION: '18.x'

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v3
      - name: Deploy
        run: ./deploy.sh
```

### Ventajas
- Fácil de empezar
- Gran comunidad
- Integración perfecta con GitHub

### Desventajas
- Vendor lock-in con GitHub
- Limitaciones en free tier
- Menos flexible que Jenkins

---

## GitLab CI/CD

### Características

- ✅ Integrado en GitLab
- ✅ 400 minutos gratis/mes
- ✅ Pipelines as code (`.gitlab-ci.yml`)
- ✅ Auto DevOps
- ✅ Built-in registry, security scanning
- ✅ Review apps automáticas

### Estructura Básica

```yaml
stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "18"

build-job:
  stage: build
  image: node:18
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/

test-job:
  stage: test
  image: node:18
  script:
    - npm install
    - npm test

deploy-job:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
```

### Ejemplo con Services

```yaml
test:
  stage: test
  image: node:18
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: testuser
    POSTGRES_PASSWORD: testpass
  script:
    - npm ci
    - npm test
```

### Ventajas
- Suite completa DevOps
- Self-hosted option
- Security scanning incluido

### Desventajas
- UI menos intuitiva
- Shared runners pueden ser lentos
- Configuración más compleja

---

## Jenkins

### Características

- ✅ Open source y gratuito
- ✅ Extremadamente flexible
- ✅ Miles de plugins
- ✅ Self-hosted (control total)
- ✅ Jenkinsfile as code
- ✅ Blue Ocean UI moderna

### Jenkinsfile Declarativo

```groovy
pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }
    
    post {
        always {
            junit 'test-results/*.xml'
            cleanWs()
        }
        failure {
            mail to: 'team@example.com',
                 subject: "Build Failed: ${env.JOB_NAME}",
                 body: "Check ${env.BUILD_URL}"
        }
    }
}
```

### Ventajas
- Máxima flexibilidad
- No vendor lock-in
- Plugins para todo

### Desventajas
- Requiere mantenimiento servidor
- Configuración compleja
- UI anticuada (sin Blue Ocean)

---

## CircleCI

### Características

- ✅ Cloud-native
- ✅ 6000 minutos gratis/mes
- ✅ Excelente performance
- ✅ Docker layer caching
- ✅ SSH debugging
- ✅ Orbs (config reutilizable)

### Ejemplo `.circleci/config.yml`

```yaml
version: 2.1

orbs:
  node: circleci/node@5.1.0

workflows:
  build-test-deploy:
    jobs:
      - node/test:
          version: '18.0'
      - deploy:
          requires:
            - node/test
          filters:
            branches:
              only: main

jobs:
  deploy:
    docker:
      - image: cimg/node:18.0
    steps:
      - checkout
      - run:
          name: Deploy
          command: ./deploy.sh
```

### Ventajas
- Muy rápido
- Excelente UX
- Docker first-class

### Desventajas
- Precio alto en scale
- No self-hosted
- Menos features que GitLab

---

## AWS CodePipeline

### Características

- ✅ Integración nativa AWS
- ✅ Visual pipeline builder
- ✅ CodeBuild, CodeDeploy integrados
- ✅ Pay-per-use
- ✅ Multi-region deployments

### Ejemplo buildspec.yml

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
  pre_build:
    commands:
      - npm install
  build:
    commands:
      - npm run build
      - npm test
  post_build:
    commands:
      - aws s3 cp dist/ s3://my-bucket/ --recursive

artifacts:
  files:
    - '**/*'
  base-directory: dist
```

### Ventajas
- Integración perfecta AWS
- Escalabilidad ilimitada
- Multi-region built-in

### Desventajas
- Vendor lock-in AWS
- Puede ser caro
- Curva de aprendizaje

---

## Decisión: ¿Qué Plataforma Elegir?

### Flujo de Decisión

```
¿Tu código está en GitHub?
  └─ Sí → GitHub Actions (fácil, integrado)
  └─ No ↓

¿Necesitas self-hosted?
  └─ Sí → Jenkins o GitLab self-hosted
  └─ No ↓

¿Quieres all-in-one DevOps?
  └─ Sí → GitLab CI/CD
  └─ No ↓

¿Usas principalmente AWS?
  └─ Sí → AWS CodePipeline
  └─ No → CircleCI (performance) o Travis CI (simplicidad)
```

### Recomendaciones por Caso

**Startup/Proyecto nuevo:**
→ GitHub Actions o CircleCI (rápido setup)

**Enterprise con infraestructura propia:**
→ Jenkins (flexibilidad máxima)

**Equipo DevOps completo:**
→ GitLab (todo integrado)

**Infraestructura AWS:**
→ AWS CodePipeline

**Open Source project:**
→ GitHub Actions o Travis CI (free ilimitado)

---

## Migración entre Plataformas

### De Jenkins a GitHub Actions

```groovy
# Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
    }
}
```

↓ Migra a ↓

```yaml
# .github/workflows/ci.yml
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test
```

### De GitLab a GitHub Actions

Usa herramientas como `gl-to-gh-actions` converter.

---

## Mejores Prácticas Multi-Plataforma

1. **Pipeline as Code**: Siempre en VCS
2. **Modularización**: Reutiliza scripts shell/python
3. **Environment Variables**: Para configuración
4. **Secrets Management**: Nunca hardcodear
5. **Artifact Storage**: Externo cuando posible
6. **Monitoring**: Logs centralizados

---

**Próxima sección:** Automatización de Despliegues
