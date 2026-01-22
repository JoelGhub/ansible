# 2. Modelo Declarativo de Ansible

## ¿Qué es el Modelo Declarativo?

El **modelo declarativo** es un enfoque de programación en el que se describe **el estado final deseado** de los sistemas gestionados, en lugar de detallar los pasos exactos para llegar a ese estado.

En otras palabras:
- **Declarativo:** Describes **QUÉ** quieres conseguir
- **Imperativo:** Describes **CÓMO** conseguirlo paso a paso

Ansible utiliza el modelo declarativo para simplificar la gestión de infraestructuras, haciéndola más intuitiva y menos propensa a errores.

## Modelo Declarativo vs Imperativo

### Enfoque Imperativo (Scripts tradicionales)

En la programación imperativa se indican **los pasos exactos** a seguir para llegar al estado deseado.

**Ejemplo imperativo (shell script):**
```bash
#!/bin/bash

# Verificar si Apache está instalado
if ! dpkg -l | grep -q apache2; then
    echo "Instalando Apache..."
    apt-get update
    apt-get install -y apache2
else
    echo "Apache ya está instalado"
fi

# Verificar si el servicio está corriendo
if ! systemctl is-active --quiet apache2; then
    echo "Iniciando Apache..."
    systemctl start apache2
else
    echo "Apache ya está en ejecución"
fi

# Habilitar en el arranque
if ! systemctl is-enabled --quiet apache2; then
    echo "Habilitando Apache al arranque..."
    systemctl enable apache2
else
    echo "Apache ya está habilitado"
fi
```

**Problemas del enfoque imperativo:**
- Código verboso y complejo
- Necesitas manejar todos los casos y estados
- Difícil de mantener y leer
- Propenso a errores
- No es idempotente por defecto

### Enfoque Declarativo (Ansible)

En el enfoque declarativo describes **el estado deseado** y la herramienta se encarga de alcanzarlo.

**Ejemplo declarativo (Ansible):**
```yaml
---
- name: Asegurar que Apache está instalado y en ejecución
  hosts: webservers
  become: yes
  tasks:
    - name: Asegurar que Apache está instalado
      apt:
        name: apache2
        state: present
        update_cache: yes
    
    - name: Asegurar que Apache está en ejecución
      service:
        name: apache2
        state: started
        enabled: yes
```

**Ventajas del enfoque declarativo:**
- Código simple y legible
- Ansible maneja la lógica de verificación
- Fácil de entender QUÉ hace, no CÓMO
- Idempotente por diseño
- Menos líneas de código

## Idempotencia: Pilar del Modelo Declarativo

### ¿Qué es la Idempotencia?

La **idempotencia** es una propiedad matemática que significa que ejecutar la misma operación múltiples veces produce el mismo resultado que ejecutarla una sola vez.

En Ansible:
> **Ejecutar el mismo playbook múltiples veces no produce efectos secundarios si el sistema ya está en el estado deseado.**

### Ejemplo de Idempotencia

```yaml
---
- name: Ejemplo de idempotencia
  hosts: webservers
  become: yes
  tasks:
    - name: Asegurar que Apache está instalado
      apt:
        name: apache2
        state: present
```

**Primera ejecución:**
```
TASK [Asegurar que Apache está instalado] *************************************
changed: [web1]    # Apache se instaló
```

**Segunda ejecución:**
```
TASK [Asegurar que Apache está instalado] *************************************
ok: [web1]    # Apache ya estaba instalado, no se hizo nada
```

**Tercera, cuarta, quinta ejecución...**
```
TASK [Asegurar que Apache está instalado] *************************************
ok: [web1]    # Siempre el mismo resultado
```

### Ventajas de la Idempotencia

1. **Seguridad:** Puedes ejecutar playbooks repetidamente sin miedo
2. **Convergencia:** El sistema siempre converge al estado deseado
3. **Recuperación:** Puedes re-ejecutar tras un fallo parcial
4. **Mantenimiento:** Aplicar la misma configuración periódicamente
5. **Confiabilidad:** Resultados predecibles y consistentes

### Estados en Ansible

Ansible trabaja con estados definidos en los módulos:

```yaml
# Estado: paquete presente
- name: Instalar Apache
  apt:
    name: apache2
    state: present    # present, absent, latest

# Estado: servicio iniciado
- name: Iniciar Apache
  service:
    name: apache2
    state: started    # started, stopped, restarted
    enabled: yes      # Habilitar en arranque

# Estado: archivo existe
- name: Crear directorio
  file:
    path: /opt/app
    state: directory  # directory, file, link, absent

# Estado: usuario existe
- name: Crear usuario
  user:
    name: deploy
    state: present    # present, absent
```

## Gestión de la Configuración como Código (IaC)

### Infraestructura como Código (Infrastructure as Code)

Ansible permite gestionar la infraestructura usando código declarativo, lo que se conoce como **Infrastructure as Code (IaC)**.

**Beneficios de IaC:**

1. **Control de versiones:** Usa Git para versionar configuraciones
2. **Revisión de código:** Pull requests para cambios en infraestructura
3. **Documentación:** El código autodocumenta la infraestructura
4. **Reproducibilidad:** Recrea entornos idénticos fácilmente
5. **Colaboración:** Trabajo en equipo sobre la misma configuración
6. **Auditoría:** Historial completo de cambios
7. **Testing:** Probar configuraciones antes de aplicar
8. **Rollback:** Volver a versiones anteriores si es necesario

### Ejemplo de flujo IaC con Ansible

```
1. Desarrollador escribe playbook → 2. Commit a Git → 3. Pull request
                                                              ↓
                                                        4. Revisión
                                                              ↓
                                            5. Merge a rama principal
                                                              ↓
                                    6. CI/CD ejecuta playbook en staging
                                                              ↓
                                            7. Tests automáticos
                                                              ↓
                                    8. Despliegue a producción
```

### Gestión de la Configuración

La **Gestión de la configuración** es una forma de gestión de cambios en un sistema siguiendo un método que permita mantener su integridad en el tiempo.

**Características:**
- Se registran y documentan las operaciones realizadas
- Es posible saber **cuándo** y **por qué** se llevó a cabo un cambio
- Permite saber el **estado exacto** de un sistema en un momento determinado
- Facilita la **auditoría** y el **cumplimiento** normativo

## Ventajas del Modelo Declarativo

### 1. Simplicidad
El usuario se centra en el resultado final, no en los pasos intermedios.

```yaml
# Simple y claro
- name: Nginx debe estar instalado
  apt:
    name: nginx
    state: present
```

### 2. Legibilidad
Los playbooks son fáciles de leer y entender, incluso para no programadores.

```yaml
- name: Configurar servidor web
  hosts: webservers
  tasks:
    - name: Instalar Apache
      apt: name=apache2 state=present
    
    - name: Iniciar Apache
      service: name=apache2 state=started
```

### 3. Mantenibilidad
Más fácil de mantener y modificar que scripts imperativos.

```yaml
# Cambiar versión es trivial
- name: Instalar Python específico
  apt:
    name: python3.9    # Solo cambiar este valor
    state: present
```

### 4. Menos errores
No necesitas gestionar estados intermedios manualmente.

```yaml
# Ansible maneja todos los casos automáticamente
- name: Asegurar servicio en ejecución
  service:
    name: nginx
    state: started
```

### 5. Reproducibilidad
La misma configuración se puede aplicar en diferentes entornos.

```yaml
# Mismo playbook funciona en dev, staging y production
- name: Configurar aplicación
  hosts: "{{ environment }}"
  tasks:
    - name: Desplegar app
      copy: src=app.jar dest=/opt/app/
```

### 6. Convergencia al estado deseado
El sistema converge naturalmente al estado declarado.

```yaml
# Después de ejecutar, el sistema estará en este estado
- name: Estado deseado del servidor
  hosts: webservers
  tasks:
    - name: Apache instalado
      apt: name=apache2 state=present
    - name: Apache ejecutándose
      service: name=apache2 state=started
```

## Ejemplos Declarativos vs Imperativos

### Ejemplo 1: Gestión de usuarios

**Imperativo:**
```bash
#!/bin/bash
# Verificar si usuario existe
if id "deploy" &>/dev/null; then
    echo "Usuario deploy ya existe"
else
    echo "Creando usuario deploy"
    useradd -m -s /bin/bash deploy
    usermod -aG sudo deploy
fi
```

**Declarativo:**
```yaml
- name: Asegurar que usuario deploy existe
  user:
    name: deploy
    shell: /bin/bash
    groups: sudo
    state: present
```

### Ejemplo 2: Gestión de archivos

**Imperativo:**
```bash
#!/bin/bash
if [ ! -d "/opt/app" ]; then
    mkdir -p /opt/app
fi
chown ubuntu:ubuntu /opt/app
chmod 755 /opt/app
```

**Declarativo:**
```yaml
- name: Asegurar directorio de aplicación
  file:
    path: /opt/app
    state: directory
    owner: ubuntu
    group: ubuntu
    mode: '0755'
```

### Ejemplo 3: Instalación de paquetes

**Imperativo:**
```bash
#!/bin/bash
apt-get update
for pkg in nginx php mysql-server; do
    if ! dpkg -l | grep -q "^ii  $pkg"; then
        apt-get install -y $pkg
    fi
done
```

**Declarativo:**
```yaml
- name: Asegurar paquetes instalados
  apt:
    name:
      - nginx
      - php
      - mysql-server
    state: present
    update_cache: yes
```

## Casos de uso del Modelo Declarativo

1. **Configuración de servidores:** Definir el estado de servicios, paquetes, archivos
2. **Despliegue de aplicaciones:** Asegurar versiones correctas instaladas
3. **Cumplimiento normativo:** Mantener configuraciones de seguridad
4. **Disaster recovery:** Recrear infraestructura desde código
5. **Escalado:** Aplicar misma configuración a nuevos servidores
6. **Gestión de cambios:** Documentar y versionar cambios en infraestructura

## Resumen

- El **modelo declarativo** describe el estado final deseado, no los pasos
- **Imperativo:** describes CÓMO (pasos) - **Declarativo:** describes QUÉ (estado)
- La **idempotencia** permite ejecutar playbooks múltiples veces de forma segura
- Ansible maneja automáticamente la lógica de alcanzar el estado deseado
- **IaC** (Infraestructura como Código) permite versionar y colaborar
- **Gestión de configuración** documenta cambios y mantiene integridad
- Ventajas: simplicidad, legibilidad, mantenibilidad, reproducibilidad
- El modelo declarativo es fundamental para la automatización moderna

---