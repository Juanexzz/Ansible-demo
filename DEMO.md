# Guía de Demostración en Clase - Ansible Demo

Esta guía contiene la secuencia paso a paso para presentar la demostración de Ansible en clase utilizando **GitHub Codespaces** o el **Dev Container**.

---

## 0. Verificación Inicial del Entorno

Antes de comenzar la presentación, confirma que las herramientas están disponibles en la terminal:

```bash
docker --version
ansible --version
ansible-lint --version
```

---

## 1. Comandos de la Demo (Paso a Paso)

Ejecuta los siguientes comandos en orden (dentro de la carpeta `ansible-demo`):

### Paso 1: Verificación de Conectividad (Ad-hoc Ping)
Comprueba que Ansible puede comunicarse con el host de ejecución mediante el inventario local.
```bash
ansible localhost -i inventory-local.ini -m ping
```
> **Qué explicar en clase:**
> - Este es un comando *ad-hoc* de Ansible (sin necesidad de escribir un playbook).
> - El módulo `ping` de Ansible no es un ping ICMP de red: comprueba acceso local/SSH y presencia de un intérprete Python funcional.
> - La respuesta exitosa es `"ping": "pong"` con `"changed": false`.

---

### Paso 2: Verificación de Sintaxis (Syntax Check)
Valida la sintaxis YAML y la estructura del playbook sin ejecutar ninguna tarea.
```bash
ansible-playbook -i inventory-local.ini site-local.yml --syntax-check
```
> **Qué explicar en clase:**
> - Permite detectar errores tipográficos, de indentación o módulos mal declarados antes de tocar el sistema.
> - Devuelve el nombre del playbook si la sintaxis es correcta.

---

### Paso 3: Modo Simulación y Diff (Dry-Run)
Ejecuta el playbook en modo simulación (`--check`), mostrando las diferencias exactas (`--diff`) que se aplicarían en el sistema.
```bash
ansible-playbook -i inventory-local.ini site-local.yml --check --diff
```
> **Qué explicar en clase:**
> - Permite previsualizar los cambios de forma segura en entornos de producción o staging.
> - Muestra los diffs: creación del directorio `web/`, renderizado de `index.html` y creación del contenedor `miweb`.
> - **Nota técnica:** Gracias a la especificación explícita de `image: nginx:1.27` en el handler, este dry-run funciona limpiamente incluso en un entorno frío.

---

### Paso 4: Primera Ejecución (Aprovisionamiento Real)
Aplica la configuración declarada en el playbook.
```bash
ansible-playbook -i inventory-local.ini site-local.yml
```
> **Qué explicar en clase:**
> - Ansible crea el directorio de despliegue, compila la plantilla Jinja2 (`index.html.j2`), descarga la imagen `nginx:1.27` si no existe y levanta el contenedor con puertos mapeados (`8080:80`).
> - Al modificarse la plantilla, se dispara el handler `Reiniciar contenedor web`.
> - El recap final muestra: `changed=4, failed=0`.

---

### Paso 5: Segunda Ejecución (Demostración de Idempotencia)
Vuelve a ejecutar exactamente el mismo playbook sin modificar nada.
```bash
ansible-playbook -i inventory-local.ini site-local.yml
```
> **Qué explicar en clase:**
> - **Principio fundamental de la Automatización: Idempotencia.**
> - Si el sistema ya se encuentra en el estado deseado, Ansible no realiza modificaciones redundantes ni reinicia servicios innecesariamente.
> - El recap final muestra estrictamente: `changed=0, failed=0`.

---

### Paso 6: Verificación del Servicio Web
Consulta el servidor Nginx para comprobar que responde correctamente con la plantilla Jinja2 procesada.
```bash
curl -i http://localhost:8080
```
> **Qué explicar en clase:**
> - Se obtiene una respuesta HTTP 200 OK.
> - La página contiene el mensaje configurado y los facts del sistema (`ansible_facts['distribution']`, `distribution_version`) inyectados dinámicamente por Ansible.
> - En GitHub Codespaces, también puedes abrir la pestaña **Ports** y acceder al puerto 8080 desde el navegador.

---

### Paso 7: Handlers y Respuesta a Cambios
Demuestra cómo reacciona Ansible ante un cambio en la configuración.

1. Modifica temporalmente `app_message` en `site-local.yml`:
```bash
# Opción con sed:
sed -i 's/app_message: "Desplegado con Ansible"/app_message: "Desplegado con Ansible - Modificado en Clase"/' site-local.yml
```
*(O edita la línea 9 de `site-local.yml` directamente en el editor).*

2. Vuelve a ejecutar el playbook:
```bash
ansible-playbook -i inventory-local.ini site-local.yml
```
> **Qué explicar en clase:**
> - Las tareas de Docker y directorio aparecen como `ok` (no se alteraron).
> - Únicamente la tarea de la plantilla y el handler de reinicio aparecen como `changed`.
> - El recap final muestra: `changed=2, failed=0`.

3. Comprueba el cambio en vivo:
```bash
curl http://localhost:8080
```

4. Revierte el cambio a su valor original:
```bash
sed -i 's/app_message: "Desplegado con Ansible - Modificado en Clase"/app_message: "Desplegado con Ansible"/' site-local.yml
ansible-playbook -i inventory-local.ini site-local.yml
```

---

### Paso 8: Análisis Estático y Buenas Prácticas (ansible-lint)
Ejecuta la herramienta de linter oficial sobre ambos playbooks (`site.yml` y `site-local.yml`):
```bash
ansible-lint site.yml site-local.yml
```
> **Qué explicar en clase:**
> - `ansible-lint` valida convenciones de estilo, seguridad, manejo de handlers y uso adecuado de FQCN (Fully Qualified Collection Names).
> - Ambos playbooks alcanzan el perfil **production** con 0 errores y 0 advertencias.

---

## 2. Comandos para Reiniciar el Estado (Reset)

Para reiniciar completamente el laboratorio y repetir la demostración desde cero:

```bash
docker rm -f miweb && rm -rf web
```

Para verificar que el entorno quedó completamente limpio:
```bash
docker ps
ls -ld web 2>/dev/null || echo "Directorio web eliminado correctamente"
```
