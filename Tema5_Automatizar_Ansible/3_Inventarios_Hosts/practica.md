# Práctica: Creación y uso de inventarios en Ansible

## Objetivo
Aprender a crear y utilizar inventarios estáticos en formato INI y YAML, y definir variables de host y grupo.

## Pasos

### 1. Crear un inventario en formato INI
Crea un archivo llamado `inventario.ini` con el siguiente contenido:

```
[web]
192.168.56.101 ansible_user=usuario ansible_ssh_pass=contraseña
192.168.56.102

[db]
192.168.56.201

[servidores:children]
web
db
```

### 2. Crear un inventario en formato YAML
Crea un archivo llamado `inventario.yml` con el siguiente contenido:

```yaml
all:
  children:
    web:
      hosts:
        servidor1:
          ansible_host: 192.168.56.101
          ansible_user: usuario
        servidor2:
          ansible_host: 192.168.56.102
    db:
      hosts:
        db1:
          ansible_host: 192.168.56.201
```

### 3. Probar la conectividad con los hosts
Ejecuta el siguiente comando para comprobar la conexión:
```bash
ansible -i inventario.ini all -m ping
```
O usando el inventario YAML:
```bash
ansible -i inventario.yml all -m ping
```

### 4. Definir variables de grupo
Añade variables a nivel de grupo en el inventario INI:

```
[web:vars]
ansible_user=usuario
ansible_ssh_pass=contraseña
```

### 5. (Opcional) Explora inventarios dinámicos
Consulta la documentación oficial de Ansible para probar un inventario dinámico adaptado a tu entorno cloud o virtualización.

---

Esta práctica te permitirá dominar la creación y uso de inventarios en Ansible, base para cualquier automatización.