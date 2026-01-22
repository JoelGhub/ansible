# 8. Ejecución de Tareas sobre Máquinas Objetivo

## Introducción

Ansible permite ejecutar tareas de dos formas principales:

1. **Comandos ad-hoc:** Ejecutan módulos individuales sobre uno o varios hosts. Útiles para tareas rápidas, comprobaciones puntuales y operaciones one-time.

2. **Playbooks:** Ejecutan conjuntos de tareas definidas en archivos YAML. Ideales para automatización compleja, repetible y documentada.

## Comandos Ad-Hoc (Directos)

Los **comandos ad-hoc** permiten ejecutar tareas Ansible directamente desde la línea de comandos sin necesidad de escribir un playbook.

### Sintaxis básica

```bash
ansible <patrón_hosts> -m <módulo> -a "<argumentos>" [opciones]
```

**Componentes:**
- `<patrón_hosts>`: Host(s) o grupo(s) del inventario
- `-m <módulo>`: Módulo de Ansible a usar
- `-a "<argumentos>"`: Argumentos para el módulo
- `[opciones]`: Opciones adicionales (--become, -i, etc.)

### Ejemplos básicos

#### 1. Verificar conectividad (ping)

```bash
# Ping a todos los hosts
ansible all -m ping

# Ping a un grupo específico
ansible webservers -m ping

# Ping a un host específico
ansible web1.example.com -m ping
```

**Salida esperada:**
```
web1.example.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

#### 2. Obtener información del sistema

```bash
# Ver uso de disco
ansible all -a "df -h"

# Ver memoria
ansible all -a "free -m"

# Ver procesos
ansible all -a "ps aux | head -10"

# Ver uptime
ansible all -a "uptime"
```

#### 3. Operaciones con el módulo apt

```bash
# Actualizar cache de paquetes
ansible all -m apt -a "update_cache=yes" --become

# Instalar un paquete
ansible all -m apt -a "name=htop state=present" --become

# Instalar múltiples paquetes
ansible all -m apt -a "name=vim,git,curl state=present" --become

# Eliminar un paquete
ansible all -m apt -a "name=apache2 state=absent" --become

# Actualizar todos los paquetes
ansible all -m apt -a "upgrade=dist" --become
```

#### 4. Gestión de servicios

```bash
# Iniciar un servicio
ansible webservers -m service -a "name=apache2 state=started" --become

# Detener un servicio
ansible webservers -m service -a "name=apache2 state=stopped" --become

# Reiniciar un servicio
ansible webservers -m service -a "name=apache2 state=restarted" --become

# Verificar estado de un servicio
ansible webservers -m service -a "name=apache2" --become
```

#### 5. Operaciones con archivos

```bash
# Copiar archivo a hosts remotos
ansible all -m copy -a "src=/home/usuario/archivo.txt dest=/tmp/archivo.txt"

# Crear directorio
ansible all -m file -a "path=/opt/miapp state=directory mode=0755" --become

# Eliminar archivo
ansible all -m file -a "path=/tmp/archivo_viejo.txt state=absent"

# Cambiar permisos
ansible all -m file -a "path=/var/www/html owner=www-data group=www-data mode=0755" --become
```

#### 6. Gestión de usuarios

```bash
# Crear usuario
ansible all -m user -a "name=deploy shell=/bin/bash" --become

# Eliminar usuario
ansible all -m user -a "name=usuario_viejo state=absent" --become

# Añadir usuario a grupo
ansible all -m user -a "name=deploy groups=sudo append=yes" --become
```

#### 7. Recopilar información (facts)

```bash
# Obtener todos los facts
ansible all -m setup

# Filtrar facts específicos
ansible all -m setup -a "filter=ansible_distribution*"

# Solo información de red
ansible all -m setup -a "filter=ansible_default_ipv4"
```

#### 8. Ejecutar comandos shell

```bash
# Ejecutar comando simple
ansible all -m command -a "hostname"

# Ejecutar con shell (permite pipes, redirecciones, etc.)
ansible all -m shell -a "ps aux | grep apache"

# Ejecutar en directorio específico
ansible all -m shell -a "ls -la" --args "chdir=/var/www"
```

#### 9. Descargar archivos

```bash
# Descargar archivo desde URL
ansible all -m get_url -a "url=https://example.com/file.tar.gz dest=/tmp/file.tar.gz"
```

#### 10. Reiniciar servidores

```bash
# Reiniciar hosts
ansible all -a "reboot" --become

# Reiniciar con delay
ansible all -m shell -a "sleep 2 && reboot" --become
```

### Opciones importantes para comandos ad-hoc

#### `-i` o `--inventory`
Especificar archivo de inventario:
```bash
ansible all -i production/hosts.cfg -m ping
```

#### `--become`
Ejecutar con privilegios elevados (sudo):
```bash
ansible all -m apt -a "name=nginx state=present" --become
```

#### `--become-user`
Ejecutar como usuario específico:
```bash
ansible all -a "whoami" --become --become-user=www-data
```

#### `--ask-become-pass` o `-K`
Solicitar contraseña de sudo:
```bash
ansible all -m apt -a "name=vim state=present" --become -K
```

#### `-f` o `--forks`
Número de procesos paralelos:
```bash
ansible all -m ping -f 10  # Ejecutar en 10 hosts en paralelo
```

#### `--limit`
Limitar ejecución a hosts específicos:
```bash
ansible webservers -m ping --limit "web1,web2"
ansible all -m ping --limit "!databases"  # Todos excepto databases
```

#### `-v`, `-vv`, `-vvv`, `-vvvv`
Aumentar verbosidad:
```bash
ansible all -m ping -v      # Nivel 1
ansible all -m ping -vvv    # Nivel 3 (muy detallado)
```

#### `-u` o `--user`
Especificar usuario SSH:
```bash
ansible all -m ping -u ubuntu
```

#### `-k` o `--ask-pass`
Solicitar contraseña SSH:
```bash
ansible all -m ping -k
```

#### `--check`
Modo dry-run (no hace cambios reales):
```bash
ansible all -m apt -a "name=nginx state=present" --become --check
```

#### `-b` o `--background`
Ejecutar en segundo plano:
```bash
ansible all -m shell -a "long_running_script.sh" -B 3600 -P 0
```

### Casos de uso prácticos

#### Verificar conectividad de todos los servidores
```bash
ansible all -m ping
```

#### Obtener información del sistema operativo
```bash
ansible all -m setup -a "filter=ansible_distribution*"
```

#### Actualizar sistema en todos los servidores
```bash
ansible all -m apt -a "update_cache=yes upgrade=dist" --become
```

#### Copiar archivo de configuración a múltiples servidores
```bash
ansible webservers -m copy -a "src=nginx.conf dest=/etc/nginx/nginx.conf backup=yes" --become
```

#### Verificar espacio en disco
```bash
ansible all -a "df -h"
```

#### Reiniciar servicio en grupo específico
```bash
ansible webservers -m service -a "name=nginx state=restarted" --become
```

#### Crear usuarios en múltiples servidores
```bash
ansible all -m user -a "name=deploy shell=/bin/bash groups=sudo" --become
```

#### Verificar versión de paquete instalado
```bash
ansible all -m shell -a "nginx -v"
```

## Ejecución de Playbooks

Los **playbooks** son la forma recomendada para automatización compleja y repetible.

### Sintaxis básica

```bash
ansible-playbook [opciones] playbook.yml
```

### Ejemplos básicos

```bash
# Ejecutar playbook
ansible-playbook site.yml

# Con inventario específico
ansible-playbook -i production/hosts.cfg site.yml

# Con privilegios elevados
ansible-playbook site.yml --become

# Modo dry-run
ansible-playbook site.yml --check

# Con diferencias
ansible-playbook site.yml --check --diff
```

### Opciones importantes para playbooks

#### `-i` o `--inventory`
```bash
ansible-playbook -i inventario.yml playbook.yml
```

#### `--become`
```bash
ansible-playbook playbook.yml --become
```

#### `--check`
Modo simulación (no realiza cambios):
```bash
ansible-playbook playbook.yml --check
```

#### `--diff`
Muestra diferencias en archivos:
```bash
ansible-playbook playbook.yml --diff
```

#### `--limit`
Restringe ejecución a ciertos hosts:
```bash
ansible-playbook playbook.yml --limit "webservers"
ansible-playbook playbook.yml --limit "web1,web2"
```

#### `--tags`
Ejecuta solo tareas con tags específicos:
```bash
ansible-playbook playbook.yml --tags "install,configure"
```

#### `--skip-tags`
Omite tareas con tags específicos:
```bash
ansible-playbook playbook.yml --skip-tags "testing"
```

#### `--start-at-task`
Comienza desde tarea específica:
```bash
ansible-playbook playbook.yml --start-at-task "Instalar Apache"
```

#### `--step`
Ejecución paso a paso (confirma cada tarea):
```bash
ansible-playbook playbook.yml --step
```

#### `-e` o `--extra-vars`
Pasar variables:
```bash
ansible-playbook playbook.yml -e "version=2.0 environment=production"
ansible-playbook playbook.yml -e @variables.json
```

#### `--list-tasks`
Listar tareas sin ejecutar:
```bash
ansible-playbook playbook.yml --list-tasks
```

#### `--list-hosts`
Listar hosts afectados:
```bash
ansible-playbook playbook.yml --list-hosts
```

#### `--list-tags`
Listar todos los tags:
```bash
ansible-playbook playbook.yml --list-tags
```

#### `--syntax-check`
Verificar sintaxis YAML:
```bash
ansible-playbook playbook.yml --syntax-check
```

#### `-v`, `-vv`, `-vvv`, `-vvvv`
Aumentar verbosidad:
```bash
ansible-playbook playbook.yml -vvv
```

### Ejemplos avanzados

#### Ejecutar con múltiples opciones
```bash
ansible-playbook site.yml \
  -i production/hosts.cfg \
  --become \
  --check \
  --diff \
  --limit "webservers" \
  -e "app_version=2.0" \
  -vv
```

#### Ejecutar solo instalación
```bash
ansible-playbook site.yml --tags "install"
```

#### Ejecutar todo excepto testing
```bash
ansible-playbook site.yml --skip-tags "testing"
```

#### Comenzar desde tarea específica
```bash
ansible-playbook mysql.yml --become --start-at-task "Crear base de datos"
```

#### Dry-run con diferencias
```bash
ansible-playbook site.yml --check --diff -v
```

## Comparación: Ad-hoc vs Playbooks

### Cuándo usar comandos ad-hoc

✅ Tareas rápidas y puntuales
✅ Comprobaciones de conectividad
✅ Recolección de información
✅ Operaciones de emergencia
✅ Testing y exploración

**Ejemplo:**
```bash
ansible all -m ping
ansible webservers -a "df -h"
```

### Cuándo usar playbooks

✅ Automatización repetible
✅ Configuraciones complejas
✅ Múltiples tareas relacionadas
✅ Documentación de procesos
✅ Control de versiones
✅ Orquestación de servicios

**Ejemplo:**
```yaml
---
- name: Configurar servidor web completo
  hosts: webservers
  become: yes
  roles:
    - apache
    - php
    - mysql
```

## Verificación y Testing

### 1. Verificar sintaxis del playbook
```bash
ansible-playbook playbook.yml --syntax-check
```

### 2. Listar tareas que se ejecutarán
```bash
ansible-playbook playbook.yml --list-tasks
```

### 3. Listar hosts afectados
```bash
ansible-playbook playbook.yml --list-hosts
```

### 4. Dry-run (modo check)
```bash
ansible-playbook playbook.yml --check
```

### 5. Dry-run con diferencias
```bash
ansible-playbook playbook.yml --check --diff
```

### 6. Ejecutar paso a paso
```bash
ansible-playbook playbook.yml --step
```

## Debugging

### Aumentar verbosidad

```bash
# Nivel 1 - Información básica
ansible-playbook playbook.yml -v

# Nivel 2 - Información de tareas
ansible-playbook playbook.yml -vv

# Nivel 3 - Información de conexión
ansible-playbook playbook.yml -vvv

# Nivel 4 - Información de plugins
ansible-playbook playbook.yml -vvvv
```

### Ver variables de un host

```bash
ansible hostname -m debug -a "var=hostvars[inventory_hostname]"
```

### Usar módulo debug en playbook

```yaml
- name: Debug de variables
  debug:
    var: mi_variable
    verbosity: 2  # Solo se muestra con -vv
```

## Tareas ad-hoc habituales

### Mantenimiento de sistema

```bash
# Actualizar sistema
ansible all -m apt -a "update_cache=yes upgrade=dist" --become

# Limpiar paquetes
ansible all -m apt -a "autoremove=yes autoclean=yes" --become

# Verificar espacio en disco
ansible all -a "df -h"

# Verificar memoria
ansible all -a "free -m"

# Ver logs recientes
ansible all -a "journalctl -n 50" --become
```

### Gestión de servicios

```bash
# Reiniciar servicios en webservers
ansible webservers -m service -a "name=apache2 state=restarted" --become

# Verificar estado de servicios
ansible all -m service -a "name=sshd" --become

# Habilitar servicio al arranque
ansible all -m service -a "name=nginx enabled=yes" --become
```

### Recolección de información

```bash
# Información del sistema operativo
ansible all -m setup -a "filter=ansible_distribution*"

# Información de red
ansible all -m setup -a "filter=ansible_default_ipv4"

# Información de hardware
ansible all -m setup -a "filter=ansible_memtotal_mb"

# Todo
ansible all -m setup > facts.json
```

## Buenas prácticas

1. **Probar primero:** Usar comandos ad-hoc para probar antes de crear playbooks
2. **Usar --check:** Validar cambios antes de aplicar
3. **Verbosidad:** Usar -v para obtener más información cuando algo falla
4. **Limitar alcance:** Usar --limit para probar en pocos hosts primero
5. **Documentar:** Los playbooks autodocumentan, los ad-hoc no
6. **Idempotencia:** Asegurar que las operaciones se pueden repetir
7. **Backup:** Usar backup=yes al modificar archivos importantes
8. **Testing:** Probar en entornos de desarrollo/staging primero
9. **Validación:** Usar --syntax-check antes de ejecutar
10. **Monitorizar:** Revisar siempre la salida para detectar errores

## Resumen

- Los **comandos ad-hoc** son rápidos para tareas puntuales
- Los **playbooks** son mejores para automatización repetible
- Usar `--check` y `--diff` para validar cambios
- `--limit` restringe ejecución a hosts específicos
- `-v`, `-vv`, `-vvv` aumentan el nivel de detalle
- `--tags` y `--skip-tags` permiten ejecución selectiva
- `--start-at-task` permite continuar desde un punto específico
- Siempre verificar sintaxis con `--syntax-check`
- **Testing adecuado** es clave para el éxito de la automatización

---