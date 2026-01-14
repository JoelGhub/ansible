# 3. Inventarios y Hosts en Ansible


El inventario es un componente fundamental en Ansible. Es el archivo donde se definen los hosts (máquinas gestionadas) y los grupos de hosts sobre los que se ejecutarán las tareas. Permite organizar la infraestructura y segmentar los sistemas según su función, entorno o cualquier otro criterio.

## Tipos de inventario
- **Estático:** Archivo de texto plano, normalmente en formato INI o YAML. Es el más común para entornos pequeños o medianos.
- **Dinámico:** Generado automáticamente a partir de fuentes externas (cloud, scripts, CMDB, etc). Útil para infraestructuras dinámicas o en la nube.

## Formatos de inventario
- **INI:** Sencillo y directo, agrupa hosts bajo etiquetas.
- **YAML:** Más estructurado, permite definir variables y jerarquías complejas.

## Variables de host y grupo
Se pueden definir variables específicas para cada host o grupo, lo que permite personalizar la configuración y el comportamiento de los playbooks.

## Inventario dinámico
Permite a Ansible obtener la lista de hosts desde una fuente externa (AWS, Azure, GCP, scripts personalizados, etc). Es esencial para entornos cloud o con cambios frecuentes.

## Buenas prácticas
- Mantener el inventario organizado y documentado.
- Usar grupos para segmentar roles y entornos (web, db, prod, dev, etc).
- Definir variables a nivel de grupo o host según necesidad.

El inventario es la base para la ejecución eficiente y ordenada de las automatizaciones en Ansible.

## Ejemplo de inventario INI
```
[web]
192.168.1.10
192.168.1.11

group1:children
web
```

## Ejemplo de inventario YAML
```yaml
all:
  hosts:
    servidor1:
      ansible_host: 192.168.1.10
    servidor2:
      ansible_host: 192.168.1.11
```

El inventario permite organizar y agrupar los hosts para facilitar la gestión y ejecución de tareas.