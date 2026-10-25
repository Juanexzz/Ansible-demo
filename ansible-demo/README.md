# ansible-demo

Ejemplo práctico mínimo: Docker + Nginx + plantilla Jinja2 con Ansible.

## Contenido
- `ansible.cfg`, `inventory.ini`, `requirements.yml`, `site.yml`, `templates/index.html.j2`: el ejemplo del documento (requiere una VM Ubuntu por SSH).
- `inventory-local.ini` y `site-local.yml`: variante sin VM, para demostrar en el mismo equipo.
- `.devcontainer/`: deja Codespaces listo (Ansible, ansible-lint, SDK de Docker y la collection).
- `.github/workflows/ansible.yml`: extensión opcional de CI.

## Demo sin VM (GitHub Codespaces)
1. Sube esta carpeta a un repositorio de GitHub.
2. Code > Codespaces > Create codespace. Espera a que termine el postCreate.
3. En la terminal:

    ansible localhost -i inventory-local.ini -m ping
    ansible-playbook -i inventory-local.ini site-local.yml --syntax-check
    ansible-playbook -i inventory-local.ini site-local.yml --check --diff
    ansible-playbook -i inventory-local.ini site-local.yml
    ansible-playbook -i inventory-local.ini site-local.yml     # debe dar changed=0

4. Abre el puerto 8080 en la pestaña Ports para ver la página.
5. Cambia `app_message` en `site-local.yml` y vuelve a ejecutar: solo cambian la plantilla y el handler.

Nota: `--check` puede fallar en la tarea del contenedor si la imagen aún no se ha descargado; ejecútalo después de la primera corrida si ocurre.

## Demo con VM (ejemplo original)
Edita la IP en `inventory.ini` y ejecuta los comandos del documento con `site.yml`.
