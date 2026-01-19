# Ejercicio Final - Práctica Sencilla Docker

## 🎯 Objetivo

Crear una aplicación web simple con Docker que demuestre los conceptos básicos aprendidos en el curso.

## 📝 Descripción

Vas a crear **"Mi Blog Personal"**, una aplicación sencilla con:
- Una página web estática (HTML/CSS)
- Un servidor web Nginx
- Persistencia de datos con volúmenes
- Red personalizada

```
┌────────────────────────────┐
│      Mi Blog Personal      │
│                            │
│  ┌──────────────────────┐  │
│  │   Nginx (Web Server) │  │
│  │   Puerto: 8080       │  │
│  │                      │  │
│  │   Contenido HTML     │  │
│  │   Archivos CSS       │  │
│  │   Imágenes          │  │
│  └──────────────────────┘  │
│           │                │
│           ▼                │
│  ┌──────────────────────┐  │
│  │   Volumen Docker     │  │
│  │   (Persistencia)     │  │
│  └──────────────────────┘  │
└────────────────────────────┘
```

## 📂 Parte 1: Crear la Estructura

### Paso 1: Crear directorios

```bash
# Crear carpeta del proyecto
mkdir mi-blog
cd mi-blog

# Crear estructura de carpetas
mkdir -p web/css web/images
```

Deberías tener:
```
mi-blog/
├── web/
│   ├── css/
│   ├── images/
│   └── index.html (lo crearemos)
├── Dockerfile (lo crearemos)
└── docker-compose.yml (lo crearemos)
```

## 🎨 Parte 2: Crear el Contenido Web

### Paso 2: Crear la página HTML

Crea el archivo `web/index.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi Blog Personal</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <header>
        <h1>🚀 Mi Blog de Tecnología</h1>
        <p>Aprendiendo Docker paso a paso</p>
    </header>

    <main>
        <article class="post">
            <h2>¿Qué es Docker?</h2>
            <p class="date">📅 19 de Enero, 2026</p>
            <p>
                Docker es una plataforma que permite empaquetar aplicaciones 
                en contenedores. Es como una caja que contiene todo lo necesario 
                para que una aplicación funcione.
            </p>
            <div class="tags">
                <span class="tag">Docker</span>
                <span class="tag">Contenedores</span>
                <span class="tag">DevOps</span>
            </div>
        </article>

        <article class="post">
            <h2>Mis Primeros Contenedores</h2>
            <p class="date">📅 18 de Enero, 2026</p>
            <p>
                Hoy aprendí a crear mi primer contenedor Docker. ¡Es increíble 
                lo fácil que es desplegar aplicaciones! Solo necesitas un 
                Dockerfile y el comando docker build.
            </p>
            <div class="tags">
                <span class="tag">Principiantes</span>
                <span class="tag">Tutorial</span>
            </div>
        </article>

        <article class="post">
            <h2>Docker Compose</h2>
            <p class="date">📅 17 de Enero, 2026</p>
            <p>
                Docker Compose hace muy fácil gestionar aplicaciones con 
                múltiples contenedores. Con un solo archivo YAML puedes 
                definir todos los servicios que necesitas.
            </p>
            <div class="tags">
                <span class="tag">Docker Compose</span>
                <span class="tag">Orquestación</span>
            </div>
        </article>
    </main>

    <footer>
        <p>✨ Creado con Docker | Ejercicio Final del Curso</p>
    </footer>
</body>
</html>
```

### Paso 3: Crear el archivo CSS

Crea el archivo `web/css/style.css`:

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    padding: 20px;
}

header {
    background: white;
    padding: 40px;
    text-align: center;
    border-radius: 15px;
    margin-bottom: 30px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

header h1 {
    color: #667eea;
    font-size: 2.5em;
    margin-bottom: 10px;
}

header p {
    color: #666;
    font-size: 1.2em;
}

main {
    max-width: 800px;
    margin: 0 auto;
}

.post {
    background: white;
    padding: 30px;
    margin-bottom: 20px;
    border-radius: 15px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
    transition: transform 0.3s;
}

.post:hover {
    transform: translateY(-5px);
}

.post h2 {
    color: #667eea;
    margin-bottom: 10px;
    font-size: 1.8em;
}

.date {
    color: #999;
    margin-bottom: 15px;
    font-style: italic;
}

.post p {
    line-height: 1.6;
    color: #333;
    margin-bottom: 15px;
}

.tags {
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
}

.tag {
    background: #667eea;
    color: white;
    padding: 5px 15px;
    border-radius: 20px;
    font-size: 0.9em;
}

footer {
    background: white;
    padding: 20px;
    text-align: center;
    border-radius: 15px;
    margin-top: 30px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

footer p {
    color: #666;
}
```

## 🐳 Parte 3: Crear el Dockerfile

### Paso 4: Crear el Dockerfile

Crea el archivo `Dockerfile` en la raíz del proyecto:

```dockerfile
# Usar imagen base de nginx
FROM nginx:alpine

# Información del autor
LABEL maintainer="tu-email@ejemplo.com"
LABEL description="Mi blog personal con Docker"

# Copiar archivos web al contenedor
COPY web/ /usr/share/nginx/html/

# Exponer el puerto 80
EXPOSE 80

# Nginx se inicia automáticamente con la imagen
```

**Explicación línea por línea:**
- `FROM nginx:alpine`: Usa Nginx en versión ligera Alpine
- `LABEL`: Añade metadatos a la imagen
- `COPY`: Copia nuestros archivos al contenedor
- `EXPOSE`: Documenta que el contenedor escucha en el puerto 80

## 🚀 Parte 4: Construir y Ejecutar

### Paso 5: Construir la imagen

```bash
# Construir la imagen
docker build -t mi-blog:v1 .

# Ver la imagen creada
docker images | grep mi-blog
```

### Paso 6: Ejecutar el contenedor (forma básica)

```bash
# Ejecutar el contenedor
docker run -d -p 8080:80 --name mi-blog-container mi-blog:v1

# Verificar que está corriendo
docker ps

# Ver logs
docker logs mi-blog-container

# Probar en el navegador
# Abre: http://localhost:8080
```

### Paso 7: Parar y eliminar

```bash
# Parar el contenedor
docker stop mi-blog-container

# Eliminar el contenedor
docker rm mi-blog-container
```

## 📦 Parte 5: Usar Volúmenes (Persistencia)

### Paso 8: Ejecutar con volumen

Ahora vamos a usar un volumen para poder editar el contenido sin reconstruir la imagen:

```bash
# Crear un volumen
docker volume create mi-blog-data

# Ejecutar con volumen
docker run -d \
  -p 8080:80 \
  --name mi-blog-vol \
  -v mi-blog-data:/usr/share/nginx/html \
  mi-blog:v1

# Copiar archivos al volumen
docker cp web/. mi-blog-vol:/usr/share/nginx/html/

# Ver información del volumen
docker volume inspect mi-blog-data
```

**Ventaja**: Ahora puedes modificar archivos y verlos reflejados sin reiniciar.

## 🌐 Parte 6: Usar Redes

### Paso 9: Crear red personalizada

```bash
# Crear red
docker network create mi-red

# Ejecutar contenedor en la red
docker run -d \
  -p 8080:80 \
  --name mi-blog-red \
  --network mi-red \
  mi-blog:v1

# Inspeccionar la red
docker network inspect mi-red
```

## 🎼 Parte 7: Docker Compose (Recomendado)

### Paso 10: Crear docker-compose.yml

Crea el archivo `docker-compose.yml`:

```yaml
version: '3.8'

services:
  web:
    build: .
    image: mi-blog:v1
    container_name: mi-blog
    ports:
      - "8080:80"
    volumes:
      - ./web:/usr/share/nginx/html:ro
    networks:
      - blog-network
    restart: unless-stopped

networks:
  blog-network:
    driver: bridge

volumes:
  blog-data:
    driver: local
```

### Paso 11: Usar Docker Compose

```bash
# Iniciar todo
docker-compose up -d

# Ver logs
docker-compose logs -f

# Ver servicios
docker-compose ps

# Parar todo
docker-compose down

# Parar y eliminar volúmenes
docker-compose down -v
```

## ✏️ Parte 8: Modificar el Blog

### Paso 12: Añadir una nueva entrada

Edita `web/index.html` y añade un nuevo artículo:

```html
<article class="post">
    <h2>Mi Proyecto Final</h2>
    <p class="date">📅 19 de Enero, 2026</p>
    <p>
        He completado el ejercicio final del curso de Docker. 
        Ahora sé crear contenedores, usar volúmenes y redes, 
        y trabajar con Docker Compose. ¡Estoy listo para más!
    </p>
    <div class="tags">
        <span class="tag">Logro</span>
        <span class="tag">Proyecto Final</span>
    </div>
</article>
```

**Sin reconstruir nada**, recarga la página en el navegador y verás los cambios.

## 🔍 Parte 9: Inspección y Debug

### Paso 13: Comandos útiles

```bash
# Ver logs en tiempo real
docker-compose logs -f web

# Entrar al contenedor
docker-compose exec web sh

# Dentro del contenedor, ver archivos
ls -la /usr/share/nginx/html

# Salir del contenedor
exit

# Ver uso de recursos
docker stats mi-blog

# Inspeccionar el contenedor
docker inspect mi-blog

# Ver la configuración de Nginx
docker exec mi-blog cat /etc/nginx/nginx.conf
```

## 📊 Parte 10: Ejercicios Prácticos

### Ejercicio 1: Personalizar el Blog
**Tarea**: Cambia los colores del CSS a tus favoritos

**Pasos**:
1. Edita `web/css/style.css`
2. Cambia los colores (`#667eea`, `#764ba2`)
3. Recarga el navegador

### Ejercicio 2: Añadir una Imagen
**Tarea**: Añade tu foto o un logo al blog

**Pasos**:
1. Copia una imagen a `web/images/logo.png`
2. Edita `web/index.html` y añade en el header:
   ```html
   <img src="images/logo.png" alt="Logo" style="width: 100px;">
   ```
3. Recarga el navegador

### Ejercicio 3: Cambiar el Puerto
**Tarea**: Hacer que el blog funcione en el puerto 9000

**Pasos**:
1. Edita `docker-compose.yml`
2. Cambia `"8080:80"` por `"9000:80"`
3. Ejecuta: `docker-compose up -d`
4. Abre: http://localhost:9000

### Ejercicio 4: Añadir una Página "Sobre Mí"
**Tarea**: Crear una segunda página HTML

**Pasos**:
1. Crea `web/about.html` con tu información
2. En `web/index.html` añade un enlace:
   ```html
   <a href="about.html">Sobre Mí</a>
   ```

### Ejercicio 5: Optimizar la Imagen
**Tarea**: Reducir el tamaño de la imagen Docker

**Pasos**:
1. Compara tamaños: `docker images | grep mi-blog`
2. El tamaño actual debería ser ~40-50 MB (ya está optimizado con Alpine)
3. Investiga qué imágenes base son más pesadas (`nginx:latest` vs `nginx:alpine`)

## 🧹 Parte 11: Limpieza Final

```bash
# Parar y eliminar todo
docker-compose down -v

# Eliminar la imagen
docker rmi mi-blog:v1

# Verificar limpieza
docker ps -a
docker images
docker volume ls
docker network ls
```

## 📝 Entregables

Crea un documento (PDF o Word) con:

1. **Portada** con tu nombre y fecha
2. **Screenshots** de:
   - La página web funcionando en el navegador
   - Comando `docker ps` mostrando el contenedor
   - Comando `docker images` mostrando tu imagen
   - Tu página "Sobre Mí" personalizada
3. **Explicación breve** (1 página):
   - ¿Qué aprendiste?
   - ¿Qué dificultades tuviste?
   - ¿Para qué usarías Docker en el futuro?

## 🏆 Rúbrica de Evaluación

| Criterio | Puntos | Descripción |
|----------|--------|-------------|
| **Dockerfile correcto** | 20 | Imagen se construye sin errores |
| **Contenedor funciona** | 25 | Blog visible en navegador |
| **Docker Compose** | 20 | Usa docker-compose.yml correctamente |
| **Personalización** | 15 | Blog personalizado con cambios propios |
| **Documentación** | 20 | Entregable con screenshots y explicación |
| **TOTAL** | **100** | |

## ✅ Checklist de Completado

Marca lo que has logrado:

- [ ] Creé la estructura de carpetas
- [ ] Creé el archivo HTML
- [ ] Creé el archivo CSS
- [ ] Creé el Dockerfile
- [ ] Construí la imagen Docker
- [ ] Ejecuté el contenedor
- [ ] Vi el blog en el navegador
- [ ] Usé volúmenes Docker
- [ ] Creé docker-compose.yml
- [ ] Usé Docker Compose
- [ ] Personalicé el blog (colores, contenido)
- [ ] Añadí una nueva entrada
- [ ] Añadí una imagen
- [ ] Creé la página "Sobre Mí"
- [ ] Hice screenshots
- [ ] Escribí el documento de entrega

## 💡 Consejos y Trucos

### Si el puerto 8080 está ocupado:
```bash
# Ver qué usa el puerto
lsof -i :8080

# Usa otro puerto en docker-compose.yml
ports:
  - "9000:80"
```

### Si los cambios no se ven:
```bash
# Limpiar caché del navegador: Ctrl + Shift + R (o Cmd + Shift + R en Mac)

# O forzar recreación del contenedor
docker-compose up -d --force-recreate
```

### Si Docker da error de permisos:
```bash
# En Linux, añadir tu usuario al grupo docker
sudo usermod -aG docker $USER
# Luego cerrar sesión y volver a entrar
```

## 🎓 Conceptos Repasados

Con este ejercicio has practicado:

- ✅ Crear un **Dockerfile** sencillo
- ✅ **Construir** imágenes Docker
- ✅ **Ejecutar** contenedores
- ✅ Mapear **puertos** (-p)
- ✅ Usar **volúmenes** para persistencia
- ✅ Crear **redes** personalizadas
- ✅ Usar **Docker Compose**
- ✅ **Inspeccionar** contenedores
- ✅ Ver **logs**
- ✅ Modificar archivos en tiempo real

## 🚀 Siguientes Pasos

¿Quieres ir más allá? Prueba:

1. **Añadir un contador de visitas** con Redis
2. **Formulario de contacto** que guarde en un archivo
3. **Subir la imagen a Docker Hub**
4. **Usar Nginx con HTTPS** (certificado SSL)
5. **Crear un blog con backend** (Node.js o Python)

## 📚 Recursos Adicionales

- [Documentación oficial de Docker](https://docs.docker.com/)
- [Docker Hub - Repositorio de imágenes](https://hub.docker.com/)
- [Play with Docker - Práctica online](https://labs.play-with-docker.com/)

---

**¡Felicidades!** Has completado el ejercicio final. Ahora tienes las bases para usar Docker en tus proyectos. 🎉

**Recuerda**: La práctica hace al maestro. Sigue experimentando y creando proyectos con Docker.
