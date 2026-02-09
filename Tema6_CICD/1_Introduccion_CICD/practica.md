# Prácticas - Introducción a CI/CD

## Objetivo
Familiarizarte con los conceptos básicos de CI/CD mediante ejercicios prácticos que te permitirán configurar tu primer pipeline.

---

## Práctica 1: Tu Primera Integración Continua (GitHub Actions)

### Descripción
Crear un repositorio con un pipeline básico que ejecute tests automáticamente.

### Requisitos
- Cuenta de GitHub
- Git instalado localmente
- Node.js instalado (o Python/Java según preferencia)

### Pasos

#### 1. Crear el proyecto

```bash
# Crear directorio del proyecto
mkdir mi-primer-pipeline
cd mi-primer-pipeline

# Inicializar Node.js project
npm init -y

# Instalar dependencias de testing
npm install --save-dev jest

# Crear estructura básica
mkdir src tests
```

#### 2. Crear código de ejemplo

Crea el archivo `src/calculator.js`:

```javascript
class Calculator {
  add(a, b) {
    return a + b;
  }

  subtract(a, b) {
    return a - b;
  }

  multiply(a, b) {
    return a * b;
  }

  divide(a, b) {
    if (b === 0) {
      throw new Error("Cannot divide by zero");
    }
    return a / b;
  }
}

module.exports = Calculator;
```

#### 3. Crear tests

Crea el archivo `tests/calculator.test.js`:

```javascript
const Calculator = require('../src/calculator');

describe('Calculator', () => {
  let calc;

  beforeEach(() => {
    calc = new Calculator();
  });

  test('should add two numbers correctly', () => {
    expect(calc.add(2, 3)).toBe(5);
    expect(calc.add(-1, 1)).toBe(0);
    expect(calc.add(0, 0)).toBe(0);
  });

  test('should subtract two numbers correctly', () => {
    expect(calc.subtract(5, 3)).toBe(2);
    expect(calc.subtract(0, 5)).toBe(-5);
  });

  test('should multiply two numbers correctly', () => {
    expect(calc.multiply(3, 4)).toBe(12);
    expect(calc.multiply(-2, 3)).toBe(-6);
    expect(calc.multiply(0, 100)).toBe(0);
  });

  test('should divide two numbers correctly', () => {
    expect(calc.divide(10, 2)).toBe(5);
    expect(calc.divide(9, 3)).toBe(3);
  });

  test('should throw error when dividing by zero', () => {
    expect(() => calc.divide(5, 0)).toThrow("Cannot divide by zero");
  });
});
```

#### 4. Configurar Jest

Actualiza `package.json`:

```json
{
  "name": "mi-primer-pipeline",
  "version": "1.0.0",
  "description": "Proyecto de ejemplo CI/CD",
  "scripts": {
    "test": "jest",
    "test:coverage": "jest --coverage"
  },
  "keywords": ["ci", "cd", "testing"],
  "author": "Tu Nombre",
  "license": "ISC",
  "devDependencies": {
    "jest": "^29.0.0"
  }
}
```

#### 5. Probar localmente

```bash
# Ejecutar tests
npm test

# Debería ver:
# PASS  tests/calculator.test.js
#   Calculator
#     ✓ should add two numbers correctly (2 ms)
#     ✓ should subtract two numbers correctly
#     ✓ should multiply two numbers correctly (1 ms)
#     ✓ should divide two numbers correctly
#     ✓ should throw error when dividing by zero (1 ms)
# 
# Test Suites: 1 passed, 1 total
# Tests:       5 passed, 5 total
```

#### 6. Crear el workflow de GitHub Actions

Crea el archivo `.github/workflows/ci.yml`:

```yaml
name: CI Pipeline

# Trigger: cuando hay push o PR a main
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    
    steps:
      # 1. Checkout del código
      - name: Checkout code
        uses: actions/checkout@v3
      
      # 2. Setup Node.js
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      # 3. Instalar dependencias
      - name: Install dependencies
        run: npm ci
      
      # 4. Ejecutar tests
      - name: Run tests
        run: npm test
      
      # 5. Generar coverage report
      - name: Generate coverage report
        run: npm run test:coverage
      
      # 6. Upload coverage (opcional)
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        if: matrix.node-version == '18.x'
        with:
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella
```

#### 7. Inicializar Git y subir a GitHub

```bash
# Inicializar repositorio
git init

# Crear .gitignore
cat > .gitignore << EOF
node_modules/
coverage/
.env
*.log
.DS_Store
EOF

# Hacer commit inicial
git add .
git commit -m "Initial commit: Setup CI pipeline with Jest"

# Crear repositorio en GitHub (hazlo desde la web)
# Luego conectar y subir:
git remote add origin https://github.com/TU_USUARIO/mi-primer-pipeline.git
git branch -M main
git push -u origin main
```

#### 8. Verificar el pipeline

1. Ve a tu repositorio en GitHub
2. Click en la pestaña "**Actions**"
3. Verás tu workflow ejecutándose
4. Click en el workflow para ver los logs en tiempo real

**Resultado esperado:**
```
✅ test (16.x) - Passed in 1m 23s
✅ test (18.x) - Passed in 1m 25s
✅ test (20.x) - Passed in 1m 21s
```

### Desafío

1. **Añade un bug intencionalmente**:
   ```javascript
   // En calculator.js, cambia:
   add(a, b) {
     return a - b; // ❌ Bug!
   }
   ```

2. **Haz commit y push**:
   ```bash
   git add .
   git commit -m "Bug: incorrect addition"
   git push
   ```

3. **Observa cómo el pipeline falla** ❌
4. **Corrige el bug** y verifica que el pipeline pasa ✅

---

## Práctica 2: Pipeline con Múltiples Etapas

### Descripción
Crear un pipeline más completo con build, test y análisis de calidad.

### Pasos

#### 1. Agregar ESLint para análisis de código

```bash
# Instalar ESLint
npm install --save-dev eslint

# Inicializar configuración
npx eslint --init

# Selecciona:
# - To check syntax, find problems, and enforce code style
# - JavaScript modules (import/export)
# - None of these (no framework)
# - Yes (TypeScript: No)
# - Node
# - Use a popular style guide → Standard
# - JavaScript
```

#### 2. Crear configuración ESLint

Crea `.eslintrc.js`:

```javascript
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
    jest: true
  },
  extends: 'standard',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  rules: {
    'semi': ['error', 'always'],
    'quotes': ['error', 'single'],
    'no-unused-vars': 'warn'
  }
};
```

#### 3. Actualizar package.json

```json
{
  "scripts": {
    "test": "jest",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/ tests/",
    "lint:fix": "eslint src/ tests/ --fix"
  }
}
```

#### 4. Actualizar el workflow

Actualiza `.github/workflows/ci.yml`:

```yaml
name: CI Pipeline Advanced

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  # Job 1: Lint
  lint:
    name: Code Quality Check
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
  
  # Job 2: Test (depende de lint)
  test:
    name: Run Tests
    needs: lint
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests with coverage
        run: npm run test:coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        if: matrix.node-version == '18.x'
  
  # Job 3: Build
  build:
    name: Build Application
    needs: [lint, test]
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: |
          echo "Building application..."
          mkdir -p dist
          cp -r src/* dist/
          echo "Build completed!"
      
      - name: Upload build artifact
        uses: actions/upload-artifact@v3
        with:
          name: build-artifact
          path: dist/
          retention-days: 7
```

#### 5. Hacer commit y ver el pipeline

```bash
git add .
git commit -m "feat: Add multi-stage pipeline with lint, test, and build"
git push
```

**Resultado esperado en GitHub Actions:**

```
Pipeline visualization:
┌──────┐
│ Lint │ ✅ (30s)
└──┬───┘
   │
   ├────────────────────┐
   ↓                    ↓
┌──────┐            ┌──────┐
│Test  │ ✅ (1m20s) │Test  │ ✅ (1m22s)
│16.x  │            │18.x  │
└──┬───┘            └──┬───┘
   │                   │
   └──────┬───────┬────┘
          │       │
        ┌─┴───────┴──┐
        │   Build    │ ✅ (45s)
        └────────────┘
```

---

## Práctica 3: Implementar Badges de Estado

### Descripción
Añadir badges en el README para mostrar el estado del pipeline.

### Pasos

#### 1. Crear README.md

```markdown
# Mi Primer Pipeline CI/CD 🚀

![CI Pipeline](https://github.com/TU_USUARIO/mi-primer-pipeline/workflows/CI%20Pipeline%20Advanced/badge.svg)
![Node Version](https://img.shields.io/badge/node-16%20%7C%2018%20%7C%2020-brightgreen)
![License](https://img.shields.io/badge/license-ISC-blue)
[![codecov](https://codecov.io/gh/TU_USUARIO/mi-primer-pipeline/branch/main/graph/badge.svg)](https://codecov.io/gh/TU_USUARIO/mi-primer-pipeline)

## 📝 Descripción

Proyecto de ejemplo para aprender CI/CD con GitHub Actions.

## ✨ Features

- ✅ Tests automáticos con Jest
- ✅ Linting con ESLint
- ✅ Coverage reports
- ✅ Multi-version testing (Node 16, 18, 20)
- ✅ Build artifacts

## 🚀 Quick Start

\`\`\`bash
# Instalar dependencias
npm install

# Ejecutar tests
npm test

# Ejecutar lint
npm run lint

# Coverage
npm run test:coverage
\`\`\`

## 📊 Coverage

La cobertura de tests actual es del **100%** 🎉

## 🔧 CI/CD Pipeline

Nuestro pipeline incluye:

1. **Lint**: Verificación de calidad de código con ESLint
2. **Test**: Ejecución de tests en Node 16, 18 y 20
3. **Build**: Generación de artefactos

## 📈 Métricas

| Métrica | Valor |
|---------|-------|
| Build time | ~2 minutos |
| Test coverage | 100% |
| Success rate | 98% |

## 🤝 Contribuir

1. Fork el proyecto
2. Crea tu feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la branch (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📄 License

ISC
```

#### 2. Commit y push

```bash
git add README.md
git commit -m "docs: Add README with pipeline badges"
git push
```

#### 3. Ver los badges en GitHub

Visita tu repositorio y verás los badges mostrando el estado del pipeline en tiempo real.

---

## Práctica 4: Implementar Branch Protection

### Descripción
Configurar protección de rama para requerir que el pipeline pase antes de hacer merge.

### Pasos

#### 1. Configurar Branch Protection en GitHub

1. Ve a tu repositorio en GitHub
2. Settings → Branches
3. Click en "**Add rule**" o "**Add branch protection rule**"
4. Branch name pattern: `main`
5. Activa estas opciones:
   - ✅ **Require a pull request before merging**
   - ✅ **Require status checks to pass before merging**
   - ✅ **Require branches to be up to date before merging**
   - Selecciona los checks: `lint`, `test`, `build`
6. Click "**Create**" o "**Save changes**"

#### 2. Crear una Pull Request

```bash
# Crear nueva rama
git checkout -b feature/new-feature

# Hacer cambios
echo "function square(x) { return x * x; }" >> src/calculator.js

# Commit
git add .
git commit -m "feat: Add square function"

# Push
git push -u origin feature/new-feature
```

#### 3. Abrir PR en GitHub

1. Ve a tu repositorio
2. Click en "**Compare & pull request**"
3. Añade descripción
4. Click en "**Create pull request**"

#### 4. Observar el proceso

- ⏳ Los checks empiezan a ejecutarse automáticamente
- ❌ No puedes hacer merge hasta que pasen
- ✅ Una vez pasen, puedes hacer merge

---

## Práctica 5: Análisis de Métricas

### Descripción
Analizar las métricas del pipeline para identificar áreas de mejora.

### Tareas

#### 1. Recopilar datos

Durante 1 semana, registra:

```yaml
Commits: 15
Pipeline runs: 18
Successful runs: 17
Failed runs: 1
Average build time: 2m 15s
Fastest build: 1m 45s
Slowest build: 3m 20s
```

#### 2. Calcular métricas

```javascript
// Success rate
const successRate = (17 / 18) * 100; // 94.4%

// Average build time (en segundos)
const avgBuildTime = 135; // 2m 15s

// Deployment frequency
const deploysPerDay = 15 / 7; // 2.14 deploys/day
```

#### 3. Crear informe

Crea `METRICS.md`:

```markdown
# Pipeline Metrics Report

**Periodo:** 1-7 Febrero 2026

## 📊 Métricas Principales

| Métrica | Valor | Objetivo | Estado |
|---------|-------|----------|--------|
| Success Rate | 94.4% | > 95% | 🟡 Cerca |
| Avg Build Time | 2m 15s | < 5m | ✅ Bueno |
| Deploy Frequency | 2.14/día | > 1/día | ✅ Bueno |
| Failed Builds | 1 | 0 | 🟡 Mejorable |

## 🎯 DORA Metrics

| Métrica | Valor | Clasificación |
|---------|-------|---------------|
| Deployment Frequency | 2.14/día | High |
| Lead Time | < 1 hora | Elite |
| MTTR | N/A | N/A |
| Change Failure Rate | 5.6% | Elite |

## 📈 Análisis

### Fortalezas
- Build time excelente (< 5 minutos)
- Alta frecuencia de despliegue
- Baja tasa de fallos

### Áreas de mejora
- Investigar el 1 build fallido
- Optimizar para llegar a 100% success rate
- Considerar paralelización para reducir build time

## 🔄 Acciones

1. ✅ Revisar logs del build fallido
2. ⏳ Implementar caché de dependencias
3. ⏳ Añadir más tests
```

---

## Práctica 6: Troubleshooting

### Descripción
Resolver problemas comunes en pipelines.

### Escenarios

#### Escenario 1: Tests Flaky

**Problema**: Test falla intermitentemente

```javascript
// tests/flaky.test.js
test('flaky test', async () => {
  const response = await fetch('https://api.example.com/random');
  const data = await response.json();
  expect(data.value).toBeGreaterThan(50); // ❌ Falla al azar
});
```

**Solución**:

```javascript
// tests/fixed.test.js
test('reliable test', async () => {
  // Mock la API call
  const mockData = { value: 75 };
  jest.spyOn(global, 'fetch').mockResolvedValue({
    json: async () => mockData
  });

  const response = await fetch('https://api.example.com/random');
  const data = await response.json();
  expect(data.value).toBeGreaterThan(50); // ✅ Siempre pasa
});
```

#### Escenario 2: Build Lento

**Problema**: Build tarda 10 minutos

**Solución 1: Caché de dependencias**

```yaml
# Ya lo tenemos con:
- uses: actions/setup-node@v3
  with:
    node-version: '18.x'
    cache: 'npm'  # ✅ Caché automático
```

**Solución 2: Paralelización**

```yaml
# Tests en paralelo (ya lo tenemos con matrix)
strategy:
  matrix:
    node-version: [16.x, 18.x, 20.x]  # ✅ Paralelo
```

#### Escenario 3: Secretos Expuestos

**Problema**: API key en el código

```javascript
// ❌ MAL
const apiKey = "sk_live_123456789";
```

**Solución**:

```javascript
// ✅ BIEN
const apiKey = process.env.API_KEY;
```

En GitHub Actions:

```yaml
steps:
  - name: Run script
    env:
      API_KEY: ${{ secrets.API_KEY }}
    run: node script.js
```

---

## Ejercicios Adicionales

### Ejercicio 1: Notificaciones

Añade notificaciones de Slack o email cuando el pipeline falla.

**Pista**: Usa `actions/slack-notify` o similar.

### Ejercicio 2: Múltiples Entornos

Crea workflows separados para `dev`, `staging`, y `production`.

### Ejercicio 3: Scheduled Pipelines

Configura un pipeline que ejecute tests cada noche.

```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM diario
```

### Ejercicio 4: Matrix Testing Avanzado

Testa múltiples OS y versiones de Node.

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [16.x, 18.x, 20.x]
```

---

## Checklist de Completitud

- [ ] ✅ Pipeline básico ejecutándose
- [ ] ✅ Tests pasando en múltiples versiones de Node
- [ ] ✅ Linting configurado
- [ ] ✅ Coverage reports generados
- [ ] ✅ Build artifacts creados
- [ ] ✅ Badges en README
- [ ] ✅ Branch protection configurado
- [ ] ✅ PR workflow funcionando
- [ ] ✅ Métricas documentadas
- [ ] ✅ Troubleshooting practicado

## Recursos para Continuar

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Jest Documentation](https://jestjs.io/)
- [ESLint Rules](https://eslint.org/docs/rules/)
- [Codecov](https://codecov.io/)

---

**¡Felicidades! Has completado tu primera implementación de CI/CD. 🎉**

Ahora estás listo para pasar a las siguientes secciones donde profundizaremos en las fases específicas de CI y CD.
