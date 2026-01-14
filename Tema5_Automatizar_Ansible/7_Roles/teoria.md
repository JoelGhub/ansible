# 7. Roles en Ansible



Los roles en Ansible son una forma avanzada de organizar y reutilizar configuraciones. Permiten agrupar tareas, variables, archivos, plantillas y handlers relacionados con una función o servicio específico, facilitando la modularidad y el mantenimiento.

## ¿Qué es Ansible Galaxy?
Ansible Galaxy es el repositorio oficial de roles y colecciones de Ansible, donde la comunidad comparte roles reutilizables para instalar, configurar y gestionar servicios y aplicaciones. Puedes buscar, descargar e instalar roles fácilmente desde https://galaxy.ansible.com.

### Comandos útiles de Ansible Galaxy
- `ansible-galaxy search <nombre>`: Buscar roles en Galaxy.
- `ansible-galaxy install <autor.rol>`: Instalar un rol desde Galaxy.
- `ansible-galaxy init <nombre_rol>`: Crear la estructura básica de un rol local.

## Estructura de un rol
Un rol tiene una estructura de carpetas predefinida:
```
rol/
  tasks/        # Tareas principales (main.yml)
  handlers/     # Handlers asociados
  files/        # Archivos estáticos
  templates/    # Plantillas Jinja2
  vars/         # Variables específicas
  defaults/     # Variables por defecto
  meta/         # Metadatos y dependencias
```

## Ventajas de los roles
- Reutilización y compartición de código.
- Mejor organización y escalabilidad.
- Facilita la colaboración y el versionado.
- Permite crear catálogos de roles reutilizables.

## Ejemplo 1: Rol local creado con Ansible Galaxy
```bash
ansible-galaxy init rol_apache
```
Esto genera la estructura básica. Edita `rol_apache/tasks/main.yml`:
```yaml
- name: Instalar Apache
  apt:
    name: apache2
    state: present
- name: Asegurar que Apache está iniciado
  service:
    name: apache2
    state: started
```
Y usa el rol en un playbook:
```yaml
- hosts: web
  become: yes
  roles:
    - rol_apache
```

## Ejemplo 2: Rol descargado de Ansible Galaxy
Instala el rol oficial de nginx:
```bash
ansible-galaxy install geerlingguy.nginx
```
Y úsalo en tu playbook:
```yaml
- hosts: web
  become: yes
  roles:
    - geerlingguy.nginx
```

## Ejemplo 3: Llamar varios roles y pasar variables
```yaml
- hosts: web
  become: yes
  roles:
    - { role: geerlingguy.nginx, nginx_vhosts: [{server_name: "miweb.local", root: "/var/www/html"}] }
    - rol_apache
```

## Buenas prácticas
- Mantener los roles independientes y bien documentados.
- Usar variables y plantillas para personalizar comportamientos.
- Compartir roles a través de Ansible Galaxy o repositorios internos.
- Versionar los roles y probarlos de forma aislada.

El uso de roles es esencial para proyectos de automatización grandes y colaborativos.
