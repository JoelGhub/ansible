# Prácticas - Monitorización y Evaluación

## Práctica 1: Dashboard de Métricas

**metrics-dashboard.sh:**
```bash
#!/bin/bash

echo "═══════════════════════════════════════"
echo "      CI/CD METRICS DASHBOARD"
echo "═══════════════════════════════════════"
echo ""

# Deployment frequency
DEPLOYS=$(gh run list --workflow=deploy.yml --limit 100 --json conclusion | jq 'length')
echo "📊 Deployments (last 30 days): $DEPLOYS"
echo "   Frequency: $(echo "scale=2; $DEPLOYS / 30" | bc)/day"
echo ""

# Success rate
SUCCESS=$(gh run list --limit 100 --json conclusion | jq '[.[] | select(.conclusion == "success")] | length')
RATE=$(echo "scale=1; $SUCCESS * 100 / 100" | bc)
echo "✅ Success Rate: ${RATE}%"
echo ""

# Average duration
AVG_DURATION=$(gh run list --limit 50 --json durationMs | jq '[.[].durationMs] | add / length / 60000')
echo "⏱️  Average Pipeline Duration: $(printf "%.1f" $AVG_DURATION) minutes"
echo ""

# Recent failures
FAILURES=$(gh run list --limit 20 --json conclusion,name | jq -r '.[] | select(.conclusion == "failure") | .name' | head -5)
if [ -n "$FAILURES" ]; then
  echo "❌ Recent Failures:"
  echo "$FAILURES"
fi
```

---

## Práctica 2: Prometheus Metrics en App

**server.js:**
```javascript
const express = require('express');
const promClient = require('prom-client');

const app = express();
const register = new promClient.Registry();

// Métricas por defecto
promClient.collectDefaultMetrics({ register });

// Métrica custom: HTTP requests
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10]
});
register.registerMetric(httpRequestDuration);

// Métrica: Total requests
const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});
register.registerMetric(httpRequestTotal);

// Middleware para medir
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration.labels(req.method, req.route?.path || req.path, res.statusCode).observe(duration);
    httpRequestTotal.labels(req.method, req.route?.path || req.path, res.statusCode).inc();
  });
  
  next();
});

// Endpoints
app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000);
```

---

## Práctica 3: DORA Metrics Calculator

**dora-metrics.js:**
```javascript
const { Octokit } = require('@octokit/rest');

const octokit = new Octokit({ auth: process.env.GITHUB_TOKEN });

async function calculateDORAMetrics(owner, repo) {
  const thirtyDaysAgo = new Date();
  thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

  // 1. Deployment Frequency
  const { data: deployments } = await octokit.repos.listDeployments({
    owner,
    repo,
    environment: 'production',
    per_page: 100
  });
  
  const recentDeploys = deployments.filter(d => 
    new Date(d.created_at) > thirtyDaysAgo
  );
  const deployFrequency = recentDeploys.length / 30;

  // 2. Lead Time (simplified)
  const { data: runs } = await octokit.actions.listWorkflowRunsForRepo({
    owner,
    repo,
    per_page: 50
  });
  
  const leadTimes = runs.workflow_runs.map(run => {
    const created = new Date(run.created_at);
    const updated = new Date(run.updated_at);
    return (updated - created) / 1000 / 60; // minutes
  });
  const avgLeadTime = leadTimes.reduce((a, b) => a + b, 0) / leadTimes.length;

  // 3. Change Failure Rate
  const failedRuns = runs.workflow_runs.filter(r => r.conclusion === 'failure');
  const failureRate = (failedRuns.length / runs.workflow_runs.length) * 100;

  // Classification
  const classifyDeployFreq = (freq) => {
    if (freq >= 1) return 'Elite/High';
    if (freq >= 0.14) return 'Medium';
    return 'Low';
  };

  const classifyLeadTime = (time) => {
    if (time < 60) return 'Elite';
    if (time < 10080) return 'High';
    if (time < 262800) return 'Medium';
    return 'Low';
  };

  const classifyFailureRate = (rate) => {
    if (rate <= 15) return 'Elite';
    if (rate <= 30) return 'High';
    if (rate <= 45) return 'Medium';
    return 'Low';
  };

  return {
    deploymentFrequency: {
      value: deployFrequency.toFixed(2) + '/day',
      classification: classifyDeployFreq(deployFrequency)
    },
    leadTime: {
      value: avgLeadTime.toFixed(0) + ' min',
      classification: classifyLeadTime(avgLeadTime)
    },
    changeFailureRate: {
      value: failureRate.toFixed(1) + '%',
      classification: classifyFailureRate(failureRate)
    }
  };
}

// Uso
calculateDORAMetrics('owner', 'repo').then(metrics => {
  console.log('📊 DORA Metrics:');
  console.log(JSON.stringify(metrics, null, 2));
});
```

---

## Práctica 4: Automated Weekly Report

**.github/workflows/weekly-report.yml:**
```yaml
name: Weekly CI/CD Report

on:
  schedule:
    - cron: '0 9 * * 1'  # Mondays 9 AM
  workflow_dispatch:

jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm install @octokit/rest
      
      - name: Generate DORA metrics
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: node dora-metrics.js > report.txt
      
      - name: Send to Slack
        uses: 8398a7/action-slack@v3
        with:
          status: custom
          custom_payload: |
            {
              text: "📊 Weekly CI/CD Report",
              attachments: [{
                color: 'good',
                text: '${{ steps.report.outputs.content }}'
              }]
            }
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Práctica 5: Performance Budget

**performance-budget.yml:**
```yaml
name: Performance Budget Check

on: [pull_request]

jobs:
  check-budget:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build
        run: npm run build
      
      - name: Check bundle size
        run: |
          SIZE=$(du -sb dist/ | cut -f1)
          MAX_SIZE=5000000  # 5 MB
          
          if [ $SIZE -gt $MAX_SIZE ]; then
            echo "❌ Bundle size ${SIZE} exceeds budget ${MAX_SIZE}"
            exit 1
          fi
          echo "✅ Bundle size OK: ${SIZE} bytes"
      
      - name: Lighthouse CI
        uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            https://staging.myapp.com
          budgetPath: ./budget.json
          uploadArtifacts: true
```

**budget.json:**
```json
{
  "budgets": [{
    "path": "/*",
    "timings": [{
      "metric": "interactive",
      "budget": 3000
    }, {
      "metric": "first-contentful-paint",
      "budget": 1000
    }],
    "resourceSizes": [{
      "resourceType": "script",
      "budget": 300
    }, {
      "resourceType": "total",
      "budget": 500
    }]
  }]
}
```

---

## Ejercicios

### Ejercicio 1: Dashboard Grafana
Crea un dashboard completo con todas las métricas DORA.

### Ejercicio 2: Alert System
Implementa alertas automáticas cuando métricas degradan.

### Ejercicio 3: A/B Testing Metrics
Mide el impacto de cambios en el pipeline (ej: caché).

### Ejercicio 4: Cost Analysis
Calcula el costo de tu pipeline CI/CD (tiempo * runners).

---

## Checklist

- [ ] Script de métricas ejecutándose
- [ ] Prometheus metrics expuestas
- [ ] DORA metrics calculadas
- [ ] Reports semanales automáticos
- [ ] Performance budget configurado
- [ ] Alertas implementadas
- [ ] Dashboard visualizado

**¡Monitorización completa! 📊✅**
