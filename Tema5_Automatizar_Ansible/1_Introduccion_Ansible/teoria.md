# 1. Introducción a Ansible

Ansible es una herramienta de automatización de TI de código abierto desarrollada por Red Hat que permite gestionar configuraciones, desplegar aplicaciones y orquestar tareas de manera sencilla y eficiente. Se basa en un enfoque sencillo, sin agentes (agentless) y utiliza SSH para comunicarse con los nodos gestionados.

## ¿Para qué sirve Ansible?
- **Gestión de configuraciones:** Mantener la configuración de servidores de forma consistente.
- **Automatización de despliegues:** Desplegar aplicaciones en múltiples entornos.
- **Orquestación de tareas:** Coordinar tareas complejas entre múltiples sistemas.
- **Provisioning:** Configurar nueva infraestructura automáticamente.
- **Gestión de parches:** Actualizar sistemas de forma masiva y controlada.

## Ventajas principales de Ansible
- **Fácil de aprender:** Sintaxis YAML legible y documentación extensa.
- **Sin agentes:** No requiere instalar software adicional en los nodos gestionados.
- **Idempotencia:** Ejecutar el mismo playbook múltiples veces produce el mismo resultado.
- **Multiplataforma:** Compatible con Linux, Windows, macOS y dispositivos de red.
- **Escalable:** Puede gestionar desde unos pocos servidores hasta miles.
- **Extensible:** Amplia biblioteca de módulos y posibilidad de crear módulos personalizados.
- **Seguro:** Usa SSH y no deja procesos en segundo plano en los nodos.

## Modelo declarativo vs. imperativo
- **Declarativo:** Se describe el estado deseado del sistema y Ansible se encarga de alcanzarlo.
- **Imperativo:** Se indican los pasos exactos a seguir para llegar al estado deseado.

Ansible utiliza principalmente el modelo declarativo, facilitando la gestión de infraestructuras complejas.

## ¿Cómo funciona Ansible?
Ansible se basa en una arquitectura simple:

- **Controlador:** Es el equipo desde el que se ejecutan los comandos de Ansible. Aquí se instalan los playbooks y la configuración.
- **Nodos gestionados:** Son los servidores o dispositivos sobre los que Ansible actúa. No requieren que se instale ningún software adicional, solo acceso SSH y Python.

El controlador se conecta a los nodos gestionados mediante SSH y ejecuta las tareas definidas en los playbooks.

## Ejemplo de flujo de trabajo detallado
1. **Preparación:** El usuario escribe un playbook en YAML definiendo tareas como instalar paquetes, copiar archivos, etc.
2. **Inventario:** Se define un archivo de inventario con la lista de servidores a gestionar.
3. **Ejecución:** El controlador ejecuta el playbook con el comando `ansible-playbook`.
4. **Conexión:** Ansible se conecta por SSH a cada nodo del inventario.
5. **Verificación:** Comprueba el estado actual del sistema en cada nodo.
6. **Aplicación:** Solo aplica cambios si es necesario (idempotencia).
7. **Informe:** Proporciona un resumen de las tareas ejecutadas y sus resultados.

## Diagrama básico de funcionamiento (ASCII)

```
       +-------------------+
       |  Controlador      |
       |  (Ansible)        |
       +-------------------+
               |
       +---------+---------+
       |         |         |
   SSH |     SSH |     SSH |
       v         v         v
 +-----------+ +-----------+ +-----------+
 | Nodo 1    | | Nodo 2    | | Nodo 3    |
 | (Servidor)| | (Servidor)| | (Servidor)|
 +-----------+ +-----------+ +-----------+
```

El controlador Ansible se conecta por SSH a los nodos gestionados para aplicar configuraciones y tareas.

## Casos de uso reales
- **Empresas de hosting:** Configurar cientos de servidores web de forma automática.
- **Desarrollo de software:** Automatizar despliegues desde desarrollo hasta producción.
- **Infraestructura como código (IaC):** Versionar y gestionar la configuración de servidores.
- **Cumplimiento normativo:** Asegurar que todos los sistemas cumplen las políticas de seguridad.
- **Recuperación ante desastres:** Recrear infraestructura rápidamente tras incidentes.

## Conceptos fundamentales
- **Playbook:** Archivo YAML que define las tareas a ejecutar.
- **Task (Tarea):** Acción específica como instalar un paquete o copiar un archivo.
- **Module (Módulo):** Componente reutilizable que ejecuta una tarea específica.
- **Inventory (Inventario):** Lista de hosts o grupos de hosts a gestionar.
- **Host:** Servidor o dispositivo individual gestionado por Ansible.
- **Facts:** Información recopilada automáticamente sobre los hosts (OS, IP, etc.).

---