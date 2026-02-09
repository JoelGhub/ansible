# TEMA 6 - Implementar Pipelines de Integración y Entrega Continua (CI/CD)

## 📋 Índice

1. [Introducción a CI/CD](./1_Introduccion_CICD/)
2. [Fases CI y CD](./2_Fases_CI_CD/)
3. [Plataformas CI/CD](./3_Plataformas_CICD/)
4. [Automatización de Despliegues](./4_Automatizacion_Despliegues/)
5. [Gestión de Secretos](./5_Gestion_Secretos/)
6. [Pruebas Post-Despliegue](./6_Pruebas_PostDespliegue/)
7. [Estrategias de Despliegue](./7_Estrategias_Despliegue/)
8. [Monitorización y Evaluación](./8_Monitorizacion_Evaluacion/)
9. [Ejercicio Final](./9_Ejercicio_Final/)

## 🎯 Objetivos del Tema

Al finalizar este tema, serás capaz de:

- ✅ Comprender los conceptos fundamentales de CI/CD y su importancia
- ✅ Explicar las diferentes fases de CI y CD con ejemplos prácticos
- ✅ Implementar pipelines automatizados en diferentes plataformas
- ✅ Automatizar despliegues tras cambios en el repositorio
- ✅ Configurar y usar secretos de forma segura
- ✅ Integrar pruebas post-despliegue en tus pipelines
- ✅ Comparar y aplicar diferentes estrategias de despliegue
- ✅ Evaluar la efectividad de tus pipelines CI/CD

## 📚 Contenido del Tema

### 1. Introducción a CI/CD
- Conceptos básicos de integración y entrega continua
- Historia y evolución de CI/CD
- Beneficios y casos de uso
- Cultura DevOps

### 2. Fases CI y CD
- Fase de Integración Continua (CI)
- Fase de Entrega Continua (CD)
- Fase de Despliegue Continuo
- Ejemplos prácticos de cada fase

### 3. Plataformas CI/CD
- GitHub Actions
- GitLab CI/CD
- Jenkins
- CircleCI
- Travis CI
- Comparativa de plataformas

### 4. Automatización de Despliegues
- Triggers automáticos
- Webhooks y eventos
- Despliegue en diferentes entornos
- Rollback automático

### 5. Gestión de Secretos
- Variables de entorno
- Secretos en GitHub Actions
- Secretos en GitLab CI
- Vault y gestores de secretos
- Buenas prácticas de seguridad

### 6. Pruebas Post-Despliegue
- Smoke tests
- Health checks
- Pruebas de integración
- Pruebas end-to-end
- Validación de despliegues

### 7. Estrategias de Despliegue
- Blue-Green Deployment
- Canary Deployment
- Rolling Deployment
- Recreate Strategy
- Shadow Deployment
- Comparativa y casos de uso

### 8. Monitorización y Evaluación
- Métricas clave de CI/CD
- DORA metrics
- Tiempo de ciclo
- Tasa de éxito de despliegues
- Mean Time To Recovery (MTTR)
- Herramientas de monitorización

### 9. Ejercicio Final
- Proyecto completo de CI/CD
- Pipeline end-to-end
- Aplicación multi-entorno
- Estrategias avanzadas

## 🛠️ Requisitos Previos

- Conocimientos básicos de Git
- Comprensión de contenedores Docker
- Familiaridad con línea de comandos
- Cuenta en GitHub/GitLab
- Conocimientos básicos de una plataforma cloud (AWS, Azure, GCP)

## 📖 Cómo Usar Este Material

1. **Teoría**: Lee los archivos `teoria.md` de cada sección para entender los conceptos
2. **Práctica**: Realiza los ejercicios de los archivos `practica.md`
3. **Ejemplos**: Consulta la carpeta `ejemplos/` para ver casos reales
4. **Ejercicio Final**: Integra todos los conocimientos en el proyecto final

## 🔗 Referencias y Recursos Adicionales

- [Martin Fowler - Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html)
- [The DevOps Handbook](https://itrevolution.com/book/the-devops-handbook/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [DORA Metrics](https://www.devops-research.com/research.html)

## 💡 Consejos para el Aprendizaje

- Practica con proyectos reales, aunque sean pequeños
- Experimenta con diferentes plataformas CI/CD
- Automatiza gradualmente, no todo a la vez
- Monitoriza y mide constantemente
- Aprende de los fallos en los pipelines
- Mantén tus pipelines simples y mantenibles

## 🆘 Solución de Problemas Comunes

### Pipeline falla aleatoriamente
- Verifica las dependencias de red
- Revisa los límites de tiempo (timeouts)
- Comprueba la disponibilidad de recursos

### Secretos expuestos
- Usa siempre variables de entorno
- Nunca hagas commit de secretos
- Configura escaneo de secretos en tu repositorio

### Despliegues lentos
- Optimiza tus contenedores
- Usa caché de dependencias
- Paraleliza tareas cuando sea posible

---

**¡Comienza tu viaje en CI/CD! 🚀**
