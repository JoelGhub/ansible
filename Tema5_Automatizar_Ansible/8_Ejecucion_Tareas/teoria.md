# 8. Ejecución de tareas sobre máquinas objetivo


Ansible permite ejecutar tareas de dos formas principales:
- **Comandos ad-hoc:** Ejecutan módulos individuales sobre uno o varios hosts, útiles para tareas rápidas o comprobaciones.
- **Playbooks:** Ejecutan conjuntos de tareas definidas en archivos YAML, ideales para automatización compleja y repetible.

## Ejecución de comandos ad-hoc
Ejemplo para comprobar conectividad:
```
ansible web -i inventario.ini -m ping
```
Ejemplo para instalar un paquete:
```
ansible web -i inventario.ini -m apt -a "name=htop state=present" --become
```

## Ejecución de playbooks
Ejecuta un playbook completo:
```
ansible-playbook -i inventario.ini playbook.yml
```

## Opciones avanzadas
- `--check`: Modo simulación (no realiza cambios).
- `--diff`: Muestra diferencias en archivos gestionados.
- `--limit`: Restringe la ejecución a ciertos hosts.
- `-v`, `-vvv`: Aumenta el nivel de detalle de la salida.

## Buenas prácticas
- Probar primero con comandos ad-hoc antes de crear playbooks.
- Usar el modo check para validar cambios.
- Revisar la salida para detectar errores o advertencias.

La correcta ejecución y verificación de tareas es clave para el éxito de la automatización con Ansible.