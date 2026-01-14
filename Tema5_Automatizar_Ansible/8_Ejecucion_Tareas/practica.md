# Práctica: Ejecución de tareas y playbooks en Ansible

## Objetivo
Aprender a ejecutar tareas ad-hoc y playbooks, y a interpretar la salida de Ansible.

## Pasos

### 1. Ejecutar un comando ad-hoc para comprobar conectividad
```bash
ansible web -i inventario.ini -m ping
```

### 2. Ejecutar un comando ad-hoc para instalar un paquete
```bash
ansible web -i inventario.ini -m apt -a "name=tree state=present" --become
```

### 3. Ejecutar un playbook
Supón que tienes un playbook llamado `instalar_apache.yml`:
```bash
ansible-playbook -i inventario.ini instalar_apache.yml
```

### 4. Usar opciones avanzadas
- Ejecuta en modo simulación:
  ```bash
  ansible-playbook -i inventario.ini instalar_apache.yml --check
  ```
- Muestra diferencias:
  ```bash
  ansible-playbook -i inventario.ini instalar_apache.yml --diff
  ```
- Limita la ejecución a un host:
  ```bash
  ansible-playbook -i inventario.ini instalar_apache.yml --limit servidor1
  ```

---

Esta práctica te permitirá ejecutar y verificar tareas de automatización con Ansible de forma profesional.