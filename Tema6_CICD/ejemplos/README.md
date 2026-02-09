# Ejemplos de Workflows CI/CD

Este directorio contiene ejemplos listos para usar de diferentes pipelines CI/CD.

---

## 📁 Contenido

### Workflows Básicos
- `simple-ci.yml` - Pipeline CI básico (lint + test)
- `docker-build.yml` - Build y push de imagen Docker
- `multi-platform.yml` - Tests en múltiples OS y versiones

### Workflows de Deploy
- `deploy-heroku.yml` - Deploy a Heroku
- `deploy-vercel.yml` - Deploy a Vercel
- `deploy-kubernetes.yml` - Deploy a Kubernetes
- `deploy-ssh.yml` - Deploy vía SSH a VPS

### Workflows Avanzados
- `blue-green.yml` - Blue-Green deployment
- `canary.yml` - Canary deployment
- `monorepo.yml` - Pipeline para monorepo
- `scheduled-tests.yml` - Tests nocturnos

### Seguridad
- `security-scan.yml` - Escaneo de vulnerabilidades
- `secret-detection.yml` - Detección de secretos

### Monitorización
- `metrics-report.yml` - Reporte semanal de métricas
- `performance-budget.yml` - Validación de performance

---

## 🚀 Cómo Usar

1. Copia el archivo `.yml` que necesites
2. Colócalo en `.github/workflows/`
3. Ajusta las variables según tu proyecto
4. Configura los secrets necesarios en GitHub
5. Push y observa la magia ✨

---

## 📚 Recursos Adicionales

- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [Awesome Actions](https://github.com/sdras/awesome-actions)
- [Action Marketplace](https://github.com/marketplace?type=actions)

---

## 💡 Ejemplos por Tecnología

### Node.js
```yaml
- uses: actions/setup-node@v3
  with:
    node-version: '18'
    cache: 'npm'
- run: npm ci
- run: npm test
```

### Python
```yaml
- uses: actions/setup-python@v4
  with:
    python-version: '3.11'
    cache: 'pip'
- run: pip install -r requirements.txt
- run: pytest
```

### Java
```yaml
- uses: actions/setup-java@v3
  with:
    java-version: '17'
    distribution: 'temurin'
    cache: 'maven'
- run: mvn clean install
```

### Go
```yaml
- uses: actions/setup-go@v4
  with:
    go-version: '1.21'
    cache: true
- run: go test ./...
```

### Docker
```yaml
- uses: docker/setup-buildx-action@v2
- uses: docker/build-push-action@v4
  with:
    push: true
    tags: myapp:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

---

## 🎯 Templates Recomendados

### Startup MVP
- CI: lint + test
- CD: auto-deploy a Vercel/Heroku
- Duración: 3-5 min

### Empresa Mediana
- CI: lint + test + security scan
- CD: deploy staging → approval → production
- Estrategia: Rolling update
- Duración: 8-12 min

### Enterprise
- CI: full test suite + security + compliance
- CD: multi-region blue-green
- Monitoring: full observability
- Duración: 15-20 min

---

¡Usa estos ejemplos como punto de partida y personalízalos según tus necesidades! 🎨
