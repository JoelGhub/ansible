# 3. Inventarios y Hosts en Ansible

## ¿Qué es un Inventario?

El **inventario** es un componente fundamental en Ansible. Es el archivo donde se definen los **hosts** (máquinas administradas) y los **grupos de hosts** sobre los que se ejecutarán las tareas. Permite organizar la infraestructura y segmentar los sistemas según su función, entorno o cualquier otro criterio.

Sin un inventario, Ansible no sabría sobre qué máquinas trabajar.

## Archivos de configuración y de inventario

En la instalación de Ansible se crean:
- **Archivo de configuración global:** `/etc/ansible/ansible.cfg`
- **Archivo de inventario global:** `/etc/ansible/hosts`

Sin embargo, es **preferible usar archivos de configuración e inventario a nivel de cada proyecto**. Esto permite usar diferentes configuraciones e inventarios en función del proyecto.

### Estructura típica de un proyecto

```
mi-proyecto/
├── ansible.cfg          # Configuración local del proyecto
├── inventario.ini       # Inventario local del proyecto
├── playbook.yml         # Playbooks del proyecto
└── roles/               # Roles del proyecto
```

## Archivo de configuración (ansible.cfg)

El archivo `ansible.cfg` permite configurar el comportamiento de Ansible para el proyecto.

**Ejemplo:**
```ini
[defaults]
inventory = ./inventario.ini  # Usar el archivo de inventario situado en la misma carpeta
host_key_checking = False     # No verificar claves SSH (útil en entornos de desarrollo)
```

> **Nota:** El nombre del archivo de inventario es completamente configurable. Puedes usar `inventario.ini`, `hosts.cfg`, `hosts.ini`, `inventory`, o cualquier nombre que prefieras, siempre que lo especifiques correctamente en `ansible.cfg`.

## Formatos de Inventario

### 1. Formato INI (más común)

El formato INI es sencillo y directo. Cada máquina aparece en una línea y es posible crear grupos de máquinas.

**Ejemplo básico:**
```ini
# Archivo hosts.cfg de inventario
20.0.1.11
20.0.1.4
```

**Ejemplo con grupos:**
```ini
# Archivo de inventario con grupos

[webservers]
web1.example.com
web2.example.com
192.168.1.10

[databases]
db1.example.com
db2.example.com

[development]
dev1.example.com
dev2.example.com
```

### 2. Grupos de grupos (grupos anidados)

Se pueden crear grupos que contengan otros grupos usando la sintaxis `:children`

**Ejemplo:**
```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

[production:children]
webservers
databases
```

### 3. Ejemplo completo con OpenStack

```ini
# Inventory hosts.cfg file

[controller]
10.0.0.51

[network]
10.0.0.52

[compute]
10.0.0.53
10.0.0.54
10.0.0.55
10.0.0.56

[block]
10.0.0.51

[shared]
10.0.0.63

[object]
10.0.0.61
10.0.0.62
```

### 4. Formato YAML

YAML permite una estructura más compleja y la definición de variables por host.

**Ejemplo:**
```yaml
all:
  hosts:
    servidor1:
      ansible_host: 192.168.1.10
      ansible_user: ubuntu
    servidor2:
      ansible_host: 192.168.1.11
      ansible_user: admin
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.20
        web2:
          ansible_host: 192.168.1.21
    databases:
      hosts:
        db1:
          ansible_host: 192.168.1.30
```

## Prueba de funcionamiento

Una vez configurado el inventario, puedes probar la conectividad con el módulo `ping`:

```bash
$ ansible all -m ping
```

**Salida esperada:**
```
20.0.1.11 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
20.0.1.4 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Variables en el inventario

Se pueden definir variables específicas para cada host o grupo.

### Variables de host

```ini
[webservers]
web1.example.com ansible_user=ubuntu ansible_port=2222
web2.example.com ansible_user=admin
```

### Variables de grupo

```ini
[webservers]
web1.example.com
web2.example.com

[webservers:vars]
ansible_user=ubuntu
ansible_port=22
http_port=80
```

## Variables de inventario útiles

- **`ansible_host`:** Dirección IP o hostname real del servidor
- **`ansible_port`:** Puerto SSH (por defecto 22)
- **`ansible_user`:** Usuario para la conexión SSH
- **`ansible_ssh_private_key_file`:** Ruta a la clave privada SSH
- **`ansible_python_interpreter`:** Ruta al intérprete Python
- **`ansible_connection`:** Tipo de conexión (ssh, local, docker, etc.)

**Ejemplo:**
```ini
[webservers]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu ansible_port=2222
web2 ansible_host=192.168.1.11 ansible_user=deploy ansible_python_interpreter=/usr/bin/python3
```

## Patrones para seleccionar hosts

Ansible permite seleccionar hosts usando patrones:

```bash
# Todos los hosts
ansible all -m ping

# Un grupo específico
ansible webservers -m ping

# Múltiples grupos
ansible webservers:databases -m ping

# Exclusión de grupos
ansible all:!databases -m ping

# Intersección de grupos
ansible webservers:&production -m ping

# Hosts específicos
ansible web1.example.com -m ping

# Usando comodines
ansible web*.example.com -m ping
```

## Inventario dinámico

En entornos cloud o con cambios frecuentes, es útil generar el inventario automáticamente desde fuentes externas.

**Características:**
- Obtiene la lista de hosts desde AWS, Azure, GCP, OpenStack, etc.
- Se actualiza automáticamente cuando cambia la infraestructura
- Puede ser un script ejecutable que devuelve JSON

**Ejemplo de uso:**
```bash
# Usar un script de inventario dinámico
ansible-playbook -i inventory_script.py playbook.yml

# Inventarios dinámicos de cloud (requieren configuración adicional)
ansible-playbook -i aws_ec2.yml playbook.yml
```

## Verificar el inventario

Ansible proporciona herramientas para verificar y visualizar el inventario:

```bash
# Listar todos los hosts
ansible all --list-hosts

# Listar hosts de un grupo específico
ansible webservers --list-hosts

# Mostrar información detallada del inventario
ansible-inventory --list

# Mostrar inventario en formato gráfico
ansible-inventory --graph
```

**Ejemplo de salida de `--graph`:**
```
@all:
  |--@ungrouped:
  |--@webservers:
  |  |--web1.example.com
  |  |--web2.example.com
  |--@databases:
  |  |--db1.example.com
  |  |--db2.example.com
```

## Buenas prácticas

1. **Organización por función:** Agrupa servidores según su rol (web, db, cache, etc.)
2. **Organización por entorno:** Separa producción, desarrollo, testing
3. **Usar nombres descriptivos:** Los nombres de grupos deben ser claros
4. **Documentar el inventario:** Añadir comentarios explicativos
5. **Variables en archivos separados:** Usar `group_vars/` y `host_vars/` en lugar de definir en el inventario
6. **Inventarios dinámicos para cloud:** Automatiza la gestión de infraestructura dinámica
7. **Control de versiones:** Mantener el inventario en Git junto con los playbooks

**Ejemplo de estructura organizada:**
```
mi-proyecto/
├── ansible.cfg
├── inventories/
│   ├── production/
│   │   ├── hosts.cfg
│   │   └── group_vars/
│   │       ├── all.yml
│   │       ├── webservers.yml
│   │       └── databases.yml
│   └── development/
│       ├── hosts.cfg
│       └── group_vars/
│           └── all.yml
└── playbooks/
    └── deploy.yml
```

## Resumen

- El **inventario** define qué hosts administra Ansible
- Se puede organizar en **grupos** para facilitar la gestión
- Admite formatos **INI** y **YAML**
- Permite definir **variables** por host o grupo
- Se pueden usar **patrones** para seleccionar hosts
- Los inventarios **dinámicos** son útiles para entornos cloud
- Es **fundamental** para la ejecución eficiente de automatizaciones

---