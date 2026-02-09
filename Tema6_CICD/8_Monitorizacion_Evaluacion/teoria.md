# Monitorización y Evaluación de CI/CD

## Introducción

La **monitorización** y **evaluación** son esenciales para medir la efectividad de tu pipeline CI/CD y identificar áreas de mejora.

---

## Métricas Clave de CI/CD

### 1. DORA Metrics (DevOps Research & Assessment)

Las **4 métricas clave** para medir performance DevOps:

#### Deployment Frequency (Frecuencia de Despliegue)

**Definición:** ¿Con qué frecuencia despliegas a producción?

```yaml
Elite: Multiple times per day
High: Between once per day and once per week
Medium: Between once per week and once per month
Low: Fewer than once per month
```

**Cómo medir:**
```bash
# Número de deploys en últimos 30 días
gh api repos/:owner/:repo/deployments \
  --jq 'map(select(.environment == "production")) | length'
```

#### Lead Time for Changes (Tiempo de Entrega)

**Definición:** Tiempo desde commit hasta producción.

```yaml
Elite: Less than one hour
High: Between one day and one week
Medium: Between one month and six months
Low: More than six months
```

**Cómo medir:**
```javascript
const leadTime = deployTime - commitTime;
```

#### Time to Restore Service (MTTR)

**Definición:** Tiempo para recuperarse de un fallo.

```yaml
Elite: Less than one hour
High: Less than one day
Medium: Between one day and one week
Low: More than one week
```

#### Change Failure Rate (Tasa de Fallos)

**Definición:** % de despliegues que requieren rollback o hotfix.

```yaml
Elite: 0-15%
High: 16-30%
Medium: 31-45%
Low: 46-60%
```

**Cómo calcular:**
```javascript
const failureRate = (failed_deploys / total_deploys) * 100;
```

---

### 2. Métricas del Pipeline

#### Build Time

**Objetivo:** < 10 minutos para feedback rápido

```yaml
- name: Measure build time
  run: |
    START=$(date +%s)
    npm run build
    END=$(date +%s)
    DURATION=$((END - START))
    echo "Build took ${DURATION}s"
    if [ $DURATION -gt 600 ]; then
      echo "⚠️ Build too slow (>10min)"
    fi
```

#### Test Pass Rate

**Objetivo:** > 95%

```javascript
const passRate = (passed_tests / total_tests) * 100;
```

#### Code Coverage

**Objetivo:** > 80%

```yaml
- name: Check coverage
  run: |
    npm test -- --coverage
    COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
    if (( $(echo "$COVERAGE < 80" | bc -l) )); then
      echo "❌ Coverage ${COVERAGE}% below 80%"
      exit 1
    fi
```

#### Pipeline Success Rate

**Objetivo:** > 90%

```bash
# Últimos 100 runs
gh run list --limit 100 --json conclusion \
  | jq '[.[] | select(.conclusion == "success")] | length'
```

---

### 3. Métricas de Aplicación

#### Error Rate

```javascript
const errorRate = (errors / total_requests) * 100;
// Objetivo: < 1%
```

#### Response Time

```yaml
p50: < 200ms
p95: < 500ms
p99: < 1000ms
```

#### Uptime

```
SLA Target: 99.9% (8.76 horas downtime/año)
```

---

## Herramientas de Monitorización

### 1. Prometheus + Grafana

**Métricas a recopilar:**

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'app'
    static_configs:
      - targets: ['app:3000']
    metrics_path: '/metrics'
```

**Implementar endpoint /metrics:**

```javascript
const promClient = require('prom-client');
const register = new promClient.Registry();

// Métricas
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

register.registerMetric(httpRequestDuration);

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

**Dashboard Grafana:**

```json
{
  "dashboard": {
    "title": "CI/CD Metrics",
    "panels": [
      {
        "title": "Deployment Frequency",
        "targets": [
          {
            "expr": "rate(deployments_total[1h])"
          }
        ]
      },
      {
        "title": "Build Duration",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, build_duration_seconds)"
          }
        ]
      }
    ]
  }
}
```

---

### 2. GitHub Actions Insights

**Acceder a métricas:**
```
Repository → Insights → Actions
```

**Métricas disponibles:**
- Workflow runs (success/failure)
- Duration trends
- Billing usage

---

### 3. DataDog / New Relic

**Integración con CI/CD:**

```yaml
- name: Send metrics to DataDog
  run: |
    curl -X POST "https://api.datadoghq.com/api/v1/series" \
      -H "Content-Type: application/json" \
      -H "DD-API-KEY: ${{ secrets.DATADOG_API_KEY }}" \
      -d '{
        "series": [{
          "metric": "cicd.deployment",
          "points": [['"$(date +%s)"', 1]],
          "tags": ["env:production", "version:'"$VERSION"'"]
        }]
      }'
```

---

## Evaluación de Efectividad

### Dashboard de CI/CD

**Ejemplo metrics.md:**

```markdown
# CI/CD Metrics Report

**Período:** Febrero 2026

## DORA Metrics

| Métrica | Valor Actual | Objetivo | Estado |
|---------|--------------|----------|--------|
| Deployment Frequency | 3.2/día | >1/día | ✅ Elite |
| Lead Time | 45 min | <1h | ✅ Elite |
| MTTR | 25 min | <1h | ✅ Elite |
| Change Failure Rate | 8% | <15% | ✅ Elite |

## Pipeline Metrics

| Métrica | Valor | Tendencia |
|---------|-------|-----------|
| Build Time | 4m 32s | ⬇️ -15% |
| Test Pass Rate | 97.8% | ⬆️ +2% |
| Coverage | 84.3% | ➡️ Stable |
| Success Rate | 94.2% | ⬆️ +3% |

## Recomendaciones

1. ✅ Mantener frecuencia de deploy
2. ⚠️ Mejorar test pass rate a >98%
3. 📊 Aumentar coverage a >85%
```

---

### Automated Reports

**generate-report.sh:**

```bash
#!/bin/bash

PERIOD_DAYS=30
OUTPUT="metrics-report.md"

echo "# CI/CD Metrics Report" > $OUTPUT
echo "**Generated:** $(date)" >> $OUTPUT
echo "" >> $OUTPUT

# Deployment frequency
DEPLOYS=$(gh api repos/:owner/:repo/deployments \
  --jq 'length')
FREQ=$(echo "scale=2; $DEPLOYS / $PERIOD_DAYS" | bc)
echo "- Deployment Frequency: ${FREQ}/day" >> $OUTPUT

# Lead time
AVG_LEAD=$(gh run list --limit 50 --json createdAt,updatedAt \
  | jq '[.[] | (.updatedAt | fromdateiso8601) - (.createdAt | fromdateiso8601)] | add / length / 60')
echo "- Avg Lead Time: ${AVG_LEAD} minutes" >> $OUTPUT

# Success rate
SUCCESS=$(gh run list --limit 100 --json conclusion \
  | jq '[.[] | select(.conclusion == "success")] | length')
TOTAL=100
RATE=$(echo "scale=2; $SUCCESS / $TOTAL * 100" | bc)
echo "- Success Rate: ${RATE}%" >> $OUTPUT

echo "" >> $OUTPUT
echo "Report generated successfully!"
```

---

### Workflow para Métricas

```yaml
name: Generate Metrics Report

on:
  schedule:
    - cron: '0 9 * * 1'  # Lunes 9 AM
  workflow_dispatch:

jobs:
  metrics:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Generate report
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          chmod +x generate-report.sh
          ./generate-report.sh
      
      - name: Commit report
        run: |
          git config user.name "CI Bot"
          git config user.email "ci@example.com"
          git add metrics-report.md
          git commit -m "chore: Update metrics report"
          git push
      
      - name: Notify team
        uses: 8398a7/action-slack@v3
        with:
          status: custom
          custom_payload: |
            {
              text: "📊 Weekly CI/CD Metrics Report generated",
              attachments: [{
                color: 'good',
                text: 'Check the repo for detailed metrics'
              }]
            }
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Mejora Continua

### Retrospectivas

**Preguntas clave:**

1. ¿Cuál fue nuestra deployment frequency este mes?
2. ¿Cuántos rollbacks tuvimos?
3. ¿Qué causó los fallos en el pipeline?
4. ¿Cómo podemos reducir el build time?
5. ¿Qué debemos mejorar el próximo sprint?

### Experimentos

```yaml
Hypothesis: Cachear node_modules reducirá build time 30%

Experiment:
  - Medir baseline: 5 min
  - Implementar caché
  - Medir nuevo: 3.5 min
  - Resultado: ✅ 30% reducción

Action: Mantener caché, aplicar a otros jobs
```

---

## Alertas

### Alertas Críticas

```yaml
- name: Alert on high failure rate
  if: failure_rate > 20%
  run: |
    curl -X POST $SLACK_WEBHOOK \
      -d '{"text":"🚨 CI/CD Failure rate above 20%!"}'

- name: Alert on slow builds
  if: build_time > 10_minutes
  run: |
    echo "⚠️ Build time exceeds 10 minutes"
```

---

## Checklist de Evaluación

- [ ] DORA metrics calculadas mensualmente
- [ ] Dashboard de métricas configurado
- [ ] Reports automáticos generándose
- [ ] Alertas configuradas para anomalías
- [ ] Retrospectivas CI/CD trimestrales
- [ ] Experimentos de mejora documentados
- [ ] Benchmark contra industry standards

---

**¡Monitorización y mejora continua! 📊**

**Próxima sección:** Ejercicio Final
