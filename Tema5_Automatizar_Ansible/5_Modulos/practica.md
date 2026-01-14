# Práctica: Uso de módulos en Ansible

## Objetivo
Familiarizarse con los módulos más comunes de Ansible y su aplicación en tareas reales.

## Pasos

### 1. Crear un playbook que use varios módulos
Crea un archivo llamado `modulos_basicos.yml` con el siguiente contenido:

```yaml
- name: Práctica de módulos básicos
  hosts: web
  become: yes
  tasks:
    - name: Crear un directorio
      file:
        path: /tmp/ejemplo_dir
        state: directory
    - name: Copiar un archivo local
      copy:
        src: ./archivo_local.txt
        dest: /tmp/ejemplo_dir/archivo_local.txt
    - name: Instalar el paquete htop
      apt:
        name: htop
        state: present
    - name: Asegurar que el servicio cron está iniciado
      service:
        name: cron
        state: started
```

### 2. Ejecutar el playbook
```bash
ansible-playbook -i inventario.ini modulos_basicos.yml
```

### 3. Explora otros módulos
Consulta la documentación oficial y prueba otros módulos como `user`, `lineinfile`, `debug`, etc.

---

Esta práctica te ayudará a dominar el uso de módulos en Ansible.