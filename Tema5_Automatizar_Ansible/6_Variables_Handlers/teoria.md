# 6. Variables y Handlers en Ansible


Las variables y handlers son herramientas clave para la reutilización y la gestión eficiente de la configuración en Ansible.

## Tipos de variables
- **Variables de playbook:** Definidas en la sección `vars` de un playbook.
- **Variables de inventario:** Definidas en el inventario, a nivel de host o grupo.
- **Variables de entorno:** Definidas en el entorno del sistema operativo.
- **Variables de facts:** Recogidas automáticamente por Ansible sobre los hosts gestionados.
- **Variables externas:** Cargadas desde archivos o sistemas externos (`vars_files`).

## Precedencia de variables
Cuando una variable se define en varios lugares, Ansible sigue un orden de precedencia para decidir cuál usar. Consulta la documentación oficial para ver la jerarquía completa.

## Uso avanzado de variables
Las variables pueden usarse en cualquier parte del playbook usando la sintaxis `{{ variable }}`. Se pueden combinar, transformar y filtrar usando Jinja2.

## Handlers
Los handlers son tareas especiales que solo se ejecutan cuando son notificadas por otras tareas. Son útiles para acciones que solo deben ocurrir si hay cambios (reiniciar servicios, recargar configuraciones, etc).

## Ejemplo avanzado de variables y handlers
```yaml
- name: Desplegar configuración avanzada
  hosts: web
  vars:
    paquete: nginx
    archivo_conf: /etc/nginx/nginx.conf
  tasks:
    - name: Instalar Nginx
      apt:
        name: "{{ paquete }}"
        state: present
      notify: Reiniciar Nginx
    - name: Copiar configuración personalizada
      copy:
        src: ./nginx.conf
        dest: "{{ archivo_conf }}"
      notify: Reiniciar Nginx
  handlers:
    - name: Reiniciar Nginx
      service:
        name: nginx
        state: restarted
```

El uso adecuado de variables y handlers permite crear playbooks flexibles, reutilizables y eficientes.
