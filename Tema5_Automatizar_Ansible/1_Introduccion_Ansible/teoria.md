# 1. Introducción a Ansible

## ¿Qué es Ansible?

Ansible es una herramienta de **automatización de TI** y **gestión de la configuración** de código abierto que permite la administración remota de sistemas mediante código. Con Ansible se pueden crear recetas para la creación de infraestructura, aprovisionamiento de recursos y despliegue de aplicaciones.

Ansible permite gestionar la configuración de decenas, cientos o miles de sistemas de forma sencilla desde cualquier lugar. Fue desarrollada inicialmente por Michael DeHaan y actualmente es mantenida por Red Hat.

La **Gestión de la configuración** es una forma de gestión de cambios en un sistema siguiendo un método que permita mantener su integridad en el tiempo. Se registran y documentan las operaciones realizadas de forma que es posible saber cuándo y por qué se llevó a cabo un cambio. Además, permite saber el estado exacto de un sistema en un momento determinado.

## ¿Para qué sirve Ansible?

- **Gestión de configuraciones:** Mantener la configuración de servidores de forma consistente.
- **Automatización de despliegues:** Desplegar aplicaciones en múltiples entornos.
- **Orquestación de tareas:** Coordinar tareas complejas entre múltiples sistemas.
- **Provisioning:** Configurar nueva infraestructura automáticamente.
- **Gestión de parches:** Actualizar sistemas de forma masiva y controlada.
- **Infraestructura como código (IaC):** La creación y administración de infraestructura como código, combinada con un sistema de control de versiones, permite el trabajo en equipo, la prueba de configuraciones, vuelta atrás a configuraciones anteriores, y otras ventajas del uso de sistemas de control de versiones.

## Ventajas principales de Ansible

### 1. No necesita agentes
- **Sin agentes (agentless):** No requiere instalar software adicional en los nodos gestionados. Esto reduce la complejidad y los puntos de fallo.
- Sólo requiere que esté instalado **Python** en las máquinas a administrar.
- Usa **SSH** para la comunicación con los nodos remotos.

### 2. Fácil de aprender y usar
- **Sintaxis YAML legible:** Los playbooks se escriben en YAML, un formato fácil de leer y escribir.
- **Documentación extensa:** Gran cantidad de documentación y comunidad activa.
- **Curva de aprendizaje suave:** Puedes empezar con comandos simples y avanzar gradualmente.

### 3. Idempotencia
- Ejecutar el mismo playbook múltiples veces produce el mismo resultado.
- Ansible solo aplica cambios si es necesario, lo que hace las operaciones seguras y predecibles.

### 4. Extensible mediante módulos
- Arquitectura basada en **módulos** que permiten extender sus capacidades casi de forma indefinida.
- Módulos para cloud (Amazon, Azure, Google, OpenStack, …)
- Virtualización (VMware, oVirt)
- Bases de datos
- Archivos y directorios
- Monitorización
- Red
- Almacenamiento
- Control de versiones
- Windows
- Y muchos más...

### 5. Otras ventajas
- **Multiplataforma:** Compatible con Linux, Windows, macOS y dispositivos de red.
- **Escalable:** Puede gestionar desde unos pocos servidores hasta miles.
- **Seguro:** Usa SSH y no deja procesos en segundo plano en los nodos.
- **Configuración sencilla:** Permite tener todo preparado para un proyecto de Ansible con rapidez.

## ¿Cómo funciona Ansible?

Ansible se basa en una arquitectura simple cliente-servidor:

### Componentes principales

- **Máquina de control (Control Node):** Es el equipo desde el que se ejecutan los comandos de Ansible. Aquí se instalan Ansible, los playbooks y la configuración. Sólo necesita tener instalado Ansible y Python.
  
- **Nodos administrados (Managed Nodes):** Son los servidores o dispositivos sobre los que Ansible actúa. **No requieren instalar Ansible**, solo necesitan:
  - Acceso SSH
  - Python 2.7+ o Python 3.5+

### Flujo de comunicación

De forma predeterminada, Ansible administra máquinas mediante el protocolo **SSH**. El controlador se conecta a los nodos administrados mediante SSH y ejecuta las tareas definidas en los playbooks.

```
   Máquina de Control              Nodos Administrados
   +------------------+            +------------------+
   |   Ansible        |   SSH      |  Servidor 1      |
   |   Python         |----------->|  Python          |
   |   Playbooks      |            +------------------+
   +------------------+            +------------------+
                      |   SSH      |  Servidor 2      |
                      |----------->|  Python          |
                      |            +------------------+
                      |            +------------------+
                      |   SSH      |  Servidor N      |
                      |----------->|  Python          |
                                   +------------------+
```

## Ejemplo de flujo de trabajo detallado

1. **Preparación:** El usuario escribe un playbook en YAML definiendo tareas como instalar paquetes, copiar archivos, configurar servicios, etc.

2. **Inventario:** Se define un archivo de inventario con la lista de servidores a gestionar (IPs o nombres de host).

3. **Ejecución:** El controlador ejecuta el playbook con el comando `ansible-playbook nombre-playbook.yml`.

4. **Conexión:** Ansible se conecta por SSH a cada nodo del inventario de forma paralela.

5. **Gathering Facts:** Ansible recopila información del sistema (opcional pero activado por defecto): nombre del host, sistema operativo, red, etc.

6. **Verificación:** Comprueba el estado actual del sistema en cada nodo.

7. **Aplicación:** Solo aplica cambios si es necesario (**idempotencia**). Si el sistema ya está en el estado deseado, no hace cambios.

8. **Informe:** Proporciona un resumen de las tareas ejecutadas y sus resultados (ok, changed, failed, skipped).

## Requisitos de Ansible

### Requisitos para la máquina de control

Actualmente Ansible puede ejecutarse desde cualquier máquina con:
- **Python 2** (versión 2.7) o **Python 3** (versiones 3.5 y posteriores)
- **Sistema operativo:** Linux, macOS, BSD
- **Nota importante:** La máquina de control **no puede ser una máquina Windows**

### Requisitos de los nodos administrados

Los nodos administrados necesitan:
- **Python 2** (versión 2.7) o **Python 3** (versiones 3.5 y posteriores)
- **Acceso SSH** habilitado
- **Usuario con privilegios** sudo (para tareas que requieran permisos de administrador)

**Nota sobre Python:** De forma predeterminada, Ansible usa el intérprete Python localizado en `/usr/bin/python`. Algunas distribuciones modernas solo tienen Python 3 (`/usr/bin/python3`). En estos casos puede producirse un error:

```
"module_stdout": "/bin/sh: /usr/bin/python: No such file or directory\r\n"
```

**Solución:** Configurar la variable de inventario `ansible_python_interpreter` para apuntar al intérprete correcto, o instalar un intérprete de Python 2.

## Instalación de Ansible

### 1. Instalación en Ubuntu/Debian (Máquina de control)

```bash
# Actualizar repositorios
sudo apt-get update

# Instalar Python
sudo apt-get install -y python-minimal

# Instalar dependencias
sudo apt-get install software-properties-common

# Añadir repositorio de Ansible
sudo apt-add-repository --yes --update ppa:ansible/ansible

# Instalar Ansible
sudo apt-get install ansible
```

### 2. Verificar la instalación

```bash
# Verificar Python
python --version
# Salida esperada: Python 2.7.12 (o superior)

# Verificar Ansible
ansible --version
# Salida esperada muestra versión, ubicación del archivo de configuración, etc.
```

Ejemplo de salida de `ansible --version`:
```
ansible 2.7.5
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ubuntu/.ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12
```

### 3. Preparación de nodos administrados

En las máquinas administradas solo necesitamos instalar Python:

```bash
sudo apt-get update
sudo apt-get install -y python-minimal
```

### 4. Configuración de claves SSH

La comunicación entre la máquina de control y las administradas es vía SSH. La máquina de control debe tener la **clave privada** y las máquinas administradas la **clave pública**.

#### Copiar clave SSH a máquinas administradas

Desde la máquina de control, copiar la clave a cada nodo administrado:

```bash
ssh-copy-id -i ~/.ssh/id_rsa usuario@IP_NODO_ADMINISTRADO
```

Ejemplo:
```bash
ssh-copy-id -i ~/.ssh/id_rsa ubuntu@20.0.0.27
ssh-copy-id -i ~/.ssh/id_rsa ubuntu@20.0.0.22
```

Después de esto, podremos conectarnos sin contraseña:
```bash
ssh ubuntu@20.0.0.27
```

### 5. Herramientas de línea de comandos instaladas con Ansible

Tras la instalación de Ansible, se instalan varias herramientas útiles:

- **`ansible`:** Permite la ejecución directa de comandos sobre un conjunto de hosts (comandos ad-hoc).
- **`ansible-playbook`:** Ejecuta playbooks sobre un conjunto de hosts.
- **`ansible-vault`:** Cifra el contenido de archivos con datos sensibles (contraseñas, claves API, etc.).
- **`ansible-galaxy`:** Instala y gestiona roles de Ansible Galaxy, una plataforma para el intercambio de roles.
- **`ansible-console`:** Consola interactiva de ejecución de comandos.
- **`ansible-config`:** Gestiona la configuración de Ansible.
- **`ansible-doc`:** Muestra documentación sobre los módulos instalados.
- **`ansible-inventory`:** Muestra información sobre el inventario de hosts.
- **`ansible-pull`:** Descarga playbooks desde un sistema de control de versiones y los ejecuta en el sistema local.

## Modelo declarativo vs. imperativo

### Enfoque Imperativo
En la programación imperativa se indican **los pasos exactos** a seguir para llegar al estado deseado.

**Ejemplo imperativo (shell script):**
```bash
apt-get update
apt-get install -y apache2
systemctl start apache2
systemctl enable apache2
```

### Enfoque Declarativo
En el enfoque declarativo se describe **el estado deseado** del sistema y la herramienta se encarga de alcanzarlo.

**Ejemplo declarativo (Ansible):**
```yaml
- name: Asegurar que Apache está instalado y en ejecución
  hosts: webservers
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present
    
    - name: Asegurar que Apache está en ejecución
      service:
        name: apache2
        state: started
        enabled: yes
```

### Ventajas del modelo declarativo

1. **Idempotencia:** Ejecutar múltiples veces no causa efectos secundarios
2. **Legibilidad:** Es más fácil entender QUÉ se quiere lograr, no CÓMO
3. **Mantenibilidad:** Cambios en el estado deseado son evidentes
4. **Menos errores:** Ansible maneja la lógica de implementación

**Ansible utiliza principalmente el modelo declarativo**, facilitando la gestión de infraestructuras complejas.

## Resumen

Ansible es una herramienta poderosa y accesible para la automatización de infraestructuras que:
- No requiere agentes en los nodos administrados
- Usa SSH para comunicación segura
- Trabaja con un modelo declarativo e idempotente
- Es fácil de aprender gracias a su sintaxis YAML
- Se puede extender con miles de módulos disponibles
- Permite gestionar desde unos pocos hasta miles de servidores

En los próximos temas profundizaremos en los archivos de configuración, inventarios, playbooks, módulos, variables, handlers y roles.

## Casos de uso reales

- **Empresas de hosting:** Configurar cientos de servidores web de forma automática.
- **Desarrollo de software:** Automatizar despliegues desde desarrollo hasta producción.
- **Infraestructura como código (IaC):** Versionar y gestionar la configuración de servidores.
- **Cumplimiento normativo:** Asegurar que todos los sistemas cumplen las políticas de seguridad.
- **Recuperación ante desastres:** Recrear infraestructura rápidamente tras incidentes.
- **Actualizaciones masivas:** Aplicar parches de seguridad a miles de servidores simultáneamente.

## Conceptos fundamentales

- **Playbook:** Archivo YAML que define las tareas a ejecutar sobre uno o varios hosts.
- **Task (Tarea):** Acción específica como instalar un paquete o copiar un archivo.
- **Module (Módulo):** Componente reutilizable que ejecuta una tarea específica (apt, copy, service, etc.).
- **Inventory (Inventario):** Lista de hosts o grupos de hosts a gestionar.
- **Host:** Servidor o dispositivo individual gestionado por Ansible.
- **Facts:** Información recopilada automáticamente sobre los hosts (OS, IP, memoria, red, etc.).
- **Role (Rol):** Agrupación de tareas, variables y archivos relacionados para reutilización.
- **Handler:** Tarea especial que se ejecuta solo cuando es notificada por otra tarea.

---