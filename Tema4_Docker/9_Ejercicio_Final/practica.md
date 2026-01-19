# Ejercicio Final - Aplicación Web con Docker

## 🎯 Objetivo del Ejercicio

Crear una **aplicación web completa** que funcione con Docker, integrando:
- ✅ Contenedores personalizados con Dockerfile
- ✅ Persistencia de datos con volúmenes
- ✅ Comunicación entre contenedores con redes
- ✅ Orquestación con Docker Compose
- ✅ Variables de entorno para configuración

## 📋 ¿Qué vas a construir?

Una aplicación de **Lista de Tareas (To-Do App)** con:

```
┌─────────────────────────────────────────────────┐
│                   TaskMaster                     │
│                                                  │
│  ┌──────────────┐      ┌──────────────┐        │
│  │   Frontend   │◄────►│   Backen┐
│        To-Do Application         │
│                                  │
│  ┌────────────────────────────┐  │
│  │   Frontend (Nginx)         │  │
│  │   Puerto: 80               │  │
│  │   HTML + CSS + JS          │  │
│  └──────────┬─────────────────┘  │
│             │                     │
│             ▼                     │
│  ┌────────────────────────────┐  │
│  │   Backend API (Node.js)    │  │
│  │   Puerto: 3000             │  │
│  └──────────┬─────────────────┘  │
│             │                     │
│             ▼                     │
│  ┌────────────────────────────┐  │
│  │   Database (PostgreSQL)    │  │
│  │   Puerto: 5432             │  │
│  │   Volumen: Persistencia    │  │
│  └────────────────────────────┘  │
└──────────────────────────────────┘
```

**3 contenedores trabajando juntos:**

| Servicio | Tecnología | Puerto | Función |
|----------|-----------|--------|---------|
| **Web** | Nginx | 80 | Mostrar la página web |
| **API** | Node.js | 3000 | Procesar peticiones y lógica |
| **Base de Datos** | PostgreSQL | 5432 | Guardar las tareas

```bash
taskmaster/
├── docker-compose.yml
├── .env
├── .dockerignore
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── src/
│       └── index.html
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── .dockerignore
│   └── src/
│       ├── index.js
│       ├── db.js
│       └── routes.js
├── database/
│   └── init.sql
├── volumes/
│   ├── postgres-data/
│   ├── redis-data/
│   └── backups/
└── README.md
```

## 🚀 Parte 1: Configuración Inicial

### 1.1 Crear el Proyecto

```bash
mkdir -p taskmaster/{frontend/src,backend/src,database,volumes/{postgres-data,redis-data,backups}}
cd taskmaster
```

### 1.2 Archivo .env

Crea el archivo `.env` con las variables de entorno:

```bash
# Base de datos
POSTGRES_USER=taskuser
POSTGRES_PASSWORD=taskpass123
POSTGRES_DB=taskmaster_db
DATABASE_HOST=database
DATABASE_PORT=5432

# Backend
NODE_ENV=production
API_PORT=3000
REDIS_HOST=cache
REDIS_PORT=6379

# Frontend
FRONTEND_PORT=80
API_URL=http://localhost:3000

# Redis
REDIS_PASSWORD=redis_secure_password
```

### 1.3 Archivo .dockerignore

```bash
# .dockerignore (en raíz del proyecto)
node_modules
npm-debug.log
.git
.gitignore
.env
*.md
.DS_Store
volumes/
*.log
.vscode
```

## 🎨 Parte 2: Frontend

### 2.1 Dockerfile del Frontend (Multi-stage)

Crea `frontend/Dockerfile`:

```dockerfile
# Etapa 1: Build
FROM node:18-alpine AS builder

WORKDIR /app

# Copiar package files
COPY package*.json ./

# Instalar dependencias
RUN npm ci --only=production

# Copiar código fuente
COPY . .

# Build de la aplicación
RUN npm run build

# Etapa 2: Producción
FROM nginx:alpine

# Copiar archivos compilados desde builder
COPY --from=builder /app/dist /usr/share/nginx/html

# Copiar configuración personalizada de nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Exponer puerto
EXPOSE 80

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

# Usuario no root
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx

USER nginx

CMD ["nginx", "-g", "daemon off;"]
```

### 2.2 Configuración de Nginx

Crea `frontend/nginx.conf`:

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    # SPA routing
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API proxy
    location /api {
        proxy_pass http://backend:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # Cache static assets
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
}
```

### 2.3 Aplicación Frontend Simple

Crea `frontend/src/index.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TaskMaster - Gestión de Tareas</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { 
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            background: white;
            border-radius: 15px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            padding: 30px;
        }
        h1 { 
            color: #667eea;
            margin-bottom: 30px;
            text-align: center;
        }
        .input-group {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        input {
            flex: 1;
            padding: 12px;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            font-size: 16px;
        }
        button {
            padding: 12px 24px;
            background: #667eea;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.3s;
        }
        button:hover { background: #5568d3; transform: translateY(-2px); }
        .task {
            background: #f8f9fa;
            padding: 15px;
            margin-bottom: 10px;
            border-radius: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            transition: all 0.3s;
        }
        .task:hover { transform: translateX(5px); box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        .task.completed { opacity: 0.6; text-decoration: line-through; }
        .task-actions button {
            margin-left: 10px;
            padding: 8px 16px;
            font-size: 14px;
        }
        .delete { background: #e74c3c; }
        .delete:hover { background: #c0392b; }
        .stats {
            display: flex;
            justify-content: space-around;
            margin-top: 30px;
            padding-top: 20px;
            border-top: 2px solid #e0e0e0;
        }
        .stat { text-align: center; }
        .stat-number { font-size: 32px; font-weight: bold; color: #667eea; }
        .stat-label { color: #999; margin-top: 5px; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 TaskMaster</h1>
        
        <div class="input-group">
            <input type="text" id="taskInput" placeholder="Escribe una nueva tarea...">
            <button onclick="addTask()">Añadir Tarea</button>
        </div>

        <div id="taskList"></div>

        <div class="stats">
            <div class="stat">
                <div class="stat-number" id="totalTasks">0</div>
                <div class="stat-label">Total</div>
            </div>
            <div class="stat">
                <div class="stat-number" id="activeTasks">0</div>
                <div class="stat-label">Activas</div>
            </div>
            <div class="stat">
                <div class="stat-number" id="completedTasks">0</div>
                <div class="stat-label">Completadas</div>
            </div>
        </div>
    </div>

    <script>
        const API_URL = '/api';

        async function loadTasks() {
            try {
                const response = await fetch(`${API_URL}/tasks`);
                const tasks = await response.json();
                displayTasks(tasks);
                updateStats(tasks);
            } catch (error) {
                console.error('Error loading tasks:', error);
            }
        }

        function displayTasks(tasks) {
            const taskList = document.getElementById('taskList');
            taskList.innerHTML = tasks.map(task => `
                <div class="task ${task.completed ? 'completed' : ''}">
                    <span>${task.title}</span>
                    <div class="task-actions">
                        <button onclick="toggleTask(${task.id}, ${!task.completed})">
                            ${task.completed ? '↩️' : '✓'}
                        </button>
                        <button class="delete" onclick="deleteTask(${task.id})">🗑️</button>
                    </div>
                </div>
            `).join('');
        }

        function updateStats(tasks) {
            document.getElementById('totalTasks').textContent = tasks.length;
            document.getElementById('activeTasks').textContent = tasks.filter(t => !t.completed).length;
            document.getElementById('completedTasks').textContent = tasks.filter(t => t.completed).length;
        }

        async function addTask() {
            const input = document.getElementById('taskInput');
            const title = input.value.trim();
            
            if (!title) return;

            try {
                await fetch(`${API_URL}/tasks`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ title })
                });
                input.value = '';
                loadTasks();
            } catch (error) {
                console.error('Error adding task:', error);
            }
        }

        async function toggleTask(id, completed) {
            try {
                await fetch(`${API_URL}/tasks/${id}`, {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ completed })
                });
                loadTasks();
            } catch (error) {
                console.error('Error updating task:', error);
            }
        }

        async function deleteTask(id) {
            try {
                await fetch(`${API_URL}/tasks/${id}`, { method: 'DELETE' });
                loadTasks();
            } catch (error) {
                console.error('Error deleting task:', error);
            }
        }

        // Cargar tareas al iniciar
        loadTasks();

        // Enter key para añadir tarea
        document.getElementById('taskInput').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') addTask();
        });
    </script>
</body>
</html>
```

### 2.4 Package.json del Frontend

Crea `frontend/package.json`:

```json
{
  "name": "taskmaster-frontend",
  "version": "1.0.0",
  "scripts": {
    "build": "mkdir -p dist && cp -r src/* dist/"
  }
}
```

## ⚙️ Parte 3: Backend API

### 3.1 Dockerfile del Backend

Crea `backend/Dockerfile`:

```dockerfile
FROM node:18-alpine

# Instalar dumb-init para manejo de señales
RUN apk add --no-cache dumb-init

# Crear usuario no privilegiado
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copiar package files
COPY package*.json ./

# Instalar dependencias
RUN npm ci --only=production && \
    npm cache clean --force

# Copiar código fuente
COPY --chown=nodejs:nodejs . .

# Cambiar a usuario no privilegiado
USER nodejs

# Exponer puerto
EXPOSE 3000

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Usar dumb-init
ENTRYPOINT ["dumb-init", "--"]

CMD ["node", "src/index.js"]
```

### 3.2 Backend .dockerignore

Crea `backend/.dockerignore`:

```
node_modules
npm-debug.log
.env
.git
*.md
.DS_Store
```

### 3.3 Package.json del Backend

Crea `backend/package.json`:

```json
{
  "name": "taskmaster-backend",
  "version": "1.0.0",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "pg": "^8.11.0",
    "redis": "^4.6.7",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3"
  }
}
```

### 3.4 Código del Backend

Crea `backend/src/index.js`:

```javascript
const express = require('express');
const cors = require('cors');
const { Pool } = require('pg');
const redis = require('redis');

const app = express();
const PORT = process.env.API_PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());

// PostgreSQL connection
const pool = new Pool({
  host: process.env.DATABASE_HOST,
  port: process.env.DATABASE_PORT,
  database: process.env.POSTGRES_DB,
  user: process.env.POSTGRES_USER,
  password: process.env.POSTGRES_PASSWORD,
});

// Redis connection
const redisClient = redis.createClient({
  socket: {
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT
  },
  password: process.env.REDIS_PASSWORD
});

redisClient.connect().catch(console.error);

// Health check endpoint
app.get('/health', async (req, res) => {
  try {
    await pool.query('SELECT 1');
    await redisClient.ping();
    res.json({ status: 'healthy', database: 'connected', cache: 'connected' });
  } catch (error) {
    res.status(500).json({ status: 'unhealthy', error: error.message });
  }
});

// Get all tasks
app.get('/api/tasks', async (req, res) => {
  try {
    // Try cache first
    const cached = await redisClient.get('tasks');
    if (cached) {
      return res.json(JSON.parse(cached));
    }

    // Query database
    const result = await pool.query(
      'SELECT * FROM tasks ORDER BY created_at DESC'
    );
    
    // Cache for 30 seconds
    await redisClient.setEx('tasks', 30, JSON.stringify(result.rows));
    
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Create task
app.post('/api/tasks', async (req, res) => {
  try {
    const { title } = req.body;
    const result = await pool.query(
      'INSERT INTO tasks (title) VALUES ($1) RETURNING *',
      [title]
    );
    
    // Invalidate cache
    await redisClient.del('tasks');
    
    res.status(201).json(result.rows[0]);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Update task
app.put('/api/tasks/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const { completed } = req.body;
    const result = await pool.query(
      'UPDATE tasks SET completed = $1 WHERE id = $2 RETURNING *',
      [completed, id]
    );
    
    // Invalidate cache
    await redisClient.del('tasks');
    
    res.json(result.rows[0]);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Delete task
app.delete('/api/tasks/:id', async (req, res) => {
  try {
    const { id } = req.params;
    await pool.query('DELETE FROM tasks WHERE id = $1', [id]);
    
    // Invalidate cache
    await redisClient.del('tasks');
    
    res.status(204).send();
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Stats endpoint
app.get('/api/stats', async (req, res) => {
  try {
    const result = await pool.query(`
      SELECT 
        COUNT(*) as total,
        SUM(CASE WHEN completed THEN 1 ELSE 0 END) as completed,
        SUM(CASE WHEN NOT completed THEN 1 ELSE 0 END) as active
      FROM tasks
    `);
    res.json(result.rows[0]);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM signal received: closing HTTP server');
  await pool.end();
  await redisClient.quit();
  process.exit(0);
});

app.listen(PORT, () => {
  console.log(`🚀 Backend API running on port ${PORT}`);
  console.log(`📊 Health check: http://localhost:${PORT}/health`);
});
```

## 🗄️ Parte 4: Base de Datos

### 4.1 Script de Inicialización

Crea `database/init.sql`:

```sql
-- Crear tabla de tareas
CREATE TABLE IF NOT EXISTS tasks (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    completed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Índices para mejor performance
CREATE INDEX idx_tasks_completed ON tasks(completed);
CREATE INDEX idx_tasks_created_at ON tasks(created_at DESC);

-- Trigger para actualizar updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_tasks_updated_at 
    BEFORE UPDATE ON tasks 
    FOR EACH ROW 
    EXECUTE FUNCTION update_updated_at_column();

-- Datos de ejemplo
INSERT INTO tasks (title, completed) VALUES
    ('Aprender Docker básico', true),
    ('Dominar Docker Compose', true),
    ('Crear Dockerfile optimizado', false),
    ('Implementar CI/CD con Docker', false),
    ('Desplegar en producción', false);

-- Vista de estadísticas
CREATE OR REPLACE VIEW task_stats AS
SELECT 
    COUNT(*) as total_tasks,
    SUM(CASE WHEN completed THEN 1 ELSE 0 END) as completed_tasks,
    SUM(CASE WHEN NOT completed THEN 1 ELSE 0 END) as active_tasks,
    ROUND(100.0 * SUM(CASE WHEN completed THEN 1 ELSE 0 END) / COUNT(*), 2) as completion_rate
FROM tasks;
```

## 🐳 Parte 5: Docker Compose

### 5.1 docker-compose.yml

Crea `docker-compose.yml` en la raíz:

```yaml
version: '3.8'

services:
  # Base de datos PostgreSQL
  database:
    image: postgres:15-alpine
    container_name: taskmaster-db
    env_file: .env
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
      - ./volumes/backups:/backups
    networks:
      - backend-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    restart: unless-stopped

  # Redis Cache
  cache:
    image: redis:7-alpine
    container_name: taskmaster-cache
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    networks:
      - backend-network
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    restart: unless-stopped

  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: taskmaster-api
    env_file: .env
    depends_on:
      database:
        condition: service_healthy
      cache:
        condition: service_healthy
    networks:
      - backend-network
      - frontend-network
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
    labels:
      - "com.taskmaster.component=api"
      - "com.taskmaster.environment=production"

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: taskmaster-web
    ports:
      - "${FRONTEND_PORT:-80}:80"
    depends_on:
      backend:
        condition: service_healthy
    networks:
      - frontend-network
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/"]
      interval: 30s
      timeout: 5s
      retries: 3
    restart: unless-stopped
    labels:
      - "com.taskmaster.component=frontend"

  # Adminer (gestor de BD opcional)
  adminer:
    image: adminer:latest
    container_name: taskmaster-adminer
    ports:
      - "8080:8080"
    networks:
      - backend-network
    depends_on:
      - database
    restart: unless-stopped
    environment:
      ADMINER_DEFAULT_SERVER: database

networks:
  backend-network:
    driver: bridge
    name: taskmaster-backend
  frontend-network:
    driver: bridge
    name: taskmaster-frontend

volumes:
  postgres-data:
    driver: local
    name: taskmaster-postgres
  redis-data:
    driver: local
    name: taskmaster-redis
```

## 🚦 Parte 6: Despliegue y Pruebas

### 6.1 Levantar la Aplicación

```bash
# Build y start
docker-compose up --build -d

# Ver logs
docker-compose logs -f

# Ver solo logs del backend
docker-compose logs -f backend

# Ver estado de los servicios
docker-compose ps
```

### 6.2 Verificar Servicios

```bash
# Health checks
curl http://localhost:3000/health

# Test API directamente
curl http://localhost:3000/api/tasks

# Crear tarea desde CLI
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Tarea desde curl"}'

# Acceder a la aplicación web
open http://localhost

# Adminer (gestor de BD)
open http://localhost:8080
# Server: database
# User: taskuser
# Password: taskpass123
# Database: taskmaster_db
```

### 6.3 Inspeccionar Recursos

```bash
# Ver redes
docker network ls
docker network inspect taskmaster-backend

# Ver volúmenes
docker volume ls
docker volume inspect taskmaster-postgres

# Estadísticas de recursos
docker stats

# Ver imágenes creadas
docker images | grep taskmaster
```

## 🔧 Parte 7: Mantenimiento y Optimización

### 7.1 Backup de la Base de Datos

```bash
# Backup manual
docker-compose exec database pg_dump -U taskuser taskmaster_db > ./volumes/backups/backup_$(date +%Y%m%d_%H%M%S).sql

# Restaurar backup
docker-compose exec -T database psql -U taskuser taskmaster_db < ./volumes/backups/backup_20260119_120000.sql
```

### 7.2 Escalar Servicios

```bash
# Escalar el backend a 3 instancias
docker-compose up -d --scale backend=3

# Ver las instancias
docker-compose ps backend
```

### 7.3 Logs y Debugging

```bash
# Logs de todos los servicios
docker-compose logs --tail=100 -f

# Entrar al contenedor del backend
docker-compose exec backend sh

# Verificar conectividad desde backend a database
docker-compose exec backend ping database

# Ver variables de entorno
docker-compose exec backend env
```

### 7.4 Actualizar un Servicio

```bash
# Modificar código del backend
# Rebuild solo el backend
docker-compose up -d --build backend

# Ver el proceso de actualización
docker-compose logs -f backend
```

## 🎯 Parte 8: Ejercicios Avanzados

### Ejercicio 1: Optimización de Imágenes
**Objetivo**: Reducir el tamaño de las imágenes

**Tareas**:
1. Analizar tamaño actual de imágenes
2. Usar multi-stage builds en el backend
3. Eliminar archivos innecesarios
4. Usar alpine como base
5. Comparar tamaños antes/después

```bash
# Ver tamaños
docker images | grep taskmaster

# Analizar capas
docker history taskmaster-backend

# Objetivo: Backend < 100MB, Frontend < 20MB
```

### Ejercicio 2: Monitorización
**Objetivo**: Añadir sistema de monitorización

**Tareas**:
1. Añadir Prometheus para métricas
2. Añadir Grafana para visualización
3. Crear dashboard personalizado
4. Configurar alertas básicas

### Ejercicio 3: Seguridad
**Objetivo**: Mejorar la seguridad del proyecto

**Tareas**:
1. Escanear imágenes con `docker scan`
2. Implementar secrets en lugar de variables en .env
3. Usar usuarios no root en TODOS los contenedores
4. Limitar recursos (CPU, memoria)
5. Configurar read-only filesystems donde sea posible

```bash
# Escanear vulnerabilidades
docker scan taskmaster-backend

# Limitar recursos
docker-compose up -d --scale backend=2 \
  --resources-preset=limited
```

### Ejercicio 4: CI/CD
**Objetivo**: Automatizar el despliegue

**Tareas**:
1. Crear GitHub Actions workflow
2. Build y push automático a Docker Hub
3. Tests automáticos
4. Deploy automático en ambiente de staging

### Ejercicio 5: Alta Disponibilidad
**Objetivo**: Hacer la aplicación resiliente

**Tareas**:
1. Configurar réplicas del backend
2. Añadir load balancer (nginx)
3. Configurar restart policies
4. Simular fallos y recovery

```bash
# Simular fallo
docker-compose stop backend

# Verificar auto-restart
docker-compose ps
```

## 📊 Parte 9: Métricas de Éxito

Al completar este ejercicio deberías tener:

| Métrica | Objetivo |
|---------|----------|
| ✅ Contenedores funcionando | 5 (frontend, backend, db, cache, adminer) |
| ✅ Tiempo de build | < 3 minutos |
| ✅ Tiempo de startup | < 30 segundos |
| ✅ Tamaño imagen backend | < 150 MB |
| ✅ Tamaño imagen frontend | < 30 MB |
| ✅ Health checks | 100% passing |
| ✅ Persistencia de datos | Sobrevive a `docker-compose down` |
| ✅ Red isolation | Frontend y backend en redes separadas |

## 🧹 Parte 10: Limpieza

```bash
# Detener todo
docker-compose down

# Detener y eliminar volúmenes (CUIDADO: pierdes datos)
docker-compose down -v

# Eliminar imágenes creadas
docker rmi $(docker images | grep taskmaster | awk '{print $3}')

# Limpieza completa de Docker
docker system prune -a --volumes
```

## 📝 Entregables

Crea un `README.md` en la raíz del proyecto que incluya:

1. **Descripción del proyecto**
2. **Arquitectura** (diagrama)
3. **Requisitos previos**
4. **Instrucciones de instalación**
5. **Comandos útiles**
6. **Troubleshooting**
7. **Screenshots** de la aplicación funcionando

## 🏆 Rúbrica de Evaluación

| Criterio | Puntos |
|----------|--------|
| Dockerfiles optimizados | 20 |
| docker-compose.yml completo | 20 |
| Redes y volúmenes configurados | 15 |
| Health checks implementados | 10 |
| Aplicación funcionando | 15 |
| Seguridad y buenas prácticas | 10 |
| Documentación (README.md) | 10 |
| **TOTAL** | **100** |

## 💡 Bonus (Extra)

- Implementar HTTPS con certificados SSL
- Añadir autenticación JWT
- Crear tests automatizados
- Implementar rate limiting
- Añadir logging centralizado (ELK Stack)
- Migrar a Kubernetes con Helm

---

## 🎓 Conclusión

Este ejercicio integra:
- ✅ Dockerfiles personalizados con multi-stage builds
- ✅ Docker Compose para orquestación
- ✅ Volúmenes para persistencia de datos
- ✅ Redes personalizadas para aislamiento
- ✅ Health checks y restart policies
- ✅ Variables de entorno y configuración
- ✅ Optimización de imágenes
- ✅ Seguridad con usuarios no root
- ✅ Stack completo (frontend, backend, database, cache)

**¡Has demostrado dominio completo de Docker!** 🚀

