# 5. Módulos de Ansible


Los módulos son el núcleo funcional de Ansible. Cada tarea de un playbook utiliza un módulo para realizar una acción concreta sobre los hosts gestionados.

## Tipos de módulos
- **Core:** Incluidos por defecto en Ansible (file, copy, yum, apt, service, user, etc).
- **Extras:** Módulos adicionales mantenidos por la comunidad.
- **Personalizados:** Puedes crear tus propios módulos en Python o cualquier lenguaje compatible.

## Argumentos y opciones
Cada módulo tiene sus propios argumentos. Por ejemplo, el módulo `file` permite crear, eliminar o modificar archivos y directorios.

## Documentación de módulos
Consulta la documentación oficial para ver todos los módulos y sus opciones:
https://docs.ansible.com/ansible/latest/collections/index.html

## Ejemplo de uso de varios módulos
```yaml
- name: Ejemplo de módulos
  hosts: web
  become: yes
  tasks:
    - name: Crear un archivo
      file:
        path: /tmp/archivo.txt
        state: touch
    - name: Copiar archivo de configuración
      copy:
        src: ./mi_config.conf
        dest: /etc/mi_config.conf
    - name: Instalar paquete curl
      apt:
        name: curl
        state: present
    - name: Asegurar que el servicio ssh está iniciado
      service:
        name: ssh
        state: started
```

## Buenas prácticas
- Usar el módulo más específico para cada tarea.
- Consultar la documentación para conocer todas las opciones.
- Probar los módulos en entornos de pruebas antes de producción.

El uso adecuado de los módulos permite automatizar cualquier aspecto de la administración de sistemas con Ansible.