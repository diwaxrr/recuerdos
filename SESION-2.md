# SESION 2 - REFACTORING FLASK KERNEL - MÓDULOS 1 Y 2 COMPLETADOS

Contexto: Continuación del refactor de app.py monolítico (~2000 líneas) a arquitectura modular tipo Kernel.

Ya habíamos superado el import circular (Sesión 1). 

En esta sesión (2), extraimos las funciones de los archivos `app1.txt` a `app4.txt` y los convertimos en módulos independientes.

REGLAS DE ORO DEL PROYECTO (Inmutables):
1. Jerarquía: Módulo -> Plugin -> Script.
2. Cada Script hace UNA SOLA tarea.
3. Ningún Script puede pasar de 150 líneas.
4. CERO importaciones desde app.py.
    * Si un script necesita la DB, importa de `core.database.connection`. 
    * Si necesita el limitador, importa de `extensions`.

VICTORIAS DE ESTA SESIÓN (Lo que ya está funcionando):\
✅ Módulo Exportar: Creado `web/contacts/export_views.py` (CSV/JSON). Probado en navegador, descarga perfecta.\
✅ Módulo Importar: Creada carpeta `web/imports/` con `csv_views.py` y `md_views.py`. 
- NOTA IMPORTANTE: La importación masiva (135k registros CSV con encabezados nuevos) se dejó pendiente a propósito para una SESION EXCLUSIVA. Habrá que usar `chunking` y `executemany` o `bulk_insert_mappings` para no quemar la RAM.

✅ Módulo Directorio: Creada carpeta `web/directory/` con `public_views.py` (directorio y micrositios /viva/), `suggestion_views.py` (JSON) y `claim_views.py` (reclamar negocio).\
✅ Limpieza de HTML: Se usarón comandos `sed` en terminal para reemplazar masivamente `url_for('legacy...')` por las rutas nuevas (`contacts.exportar`, `imports.importar_md`, `directory.directorio_publico`).

LECCIONES APRENDIDAS EN ESTA SESIÓN (Importante para el próximo Z):
1. IMPORTS CIRCULARES EN BLUEPRINTS:
    * NUNCA usar `from web.contacts import contactos_bp` dentro de una vista. 
    * Siempre usar imports relativos: `from . import contactos_bp` o `from .list_views import contactos_bp`.
2. EL PAPEL DE LEGACY.PY: 
    * Es un puente. En cuanto una ruta se migra a su nuevo blueprint (ej: directory_bp), la ruta correspondiente se DEBE BORRAR de legacy.py. Si se deja, hay conflictos de rutas duplicadas.
3. ATENCIÓN A BASE.HTML: 
	* Si una ruta nueva funciona en el código pero el navegador hace algo raro (ej: ir al index), revisar `base.html`. 
	* En esta sesión el menú global "Directorio" apuntaba a `contacts.index` en lugar de `directory.directorio_publico`.

LO QUE QUEDA POR HACER (Roadmap para la Sesión 3):

Tarea 0 (Rápida): 
- Mover la ruta `/img/<filename>` (que sirve los logos estáticos) desde donde esté ahora y registrarla en el `app.py` principal.

Módulo 3: Productos
- Crear `web/products/__init__.py` y `web/products/form_views.py`.
- Mover la ruta `/negocio/<int:id>/nuevo-producto`.
- Mover la función `save_producto_image()` a `core/utils/file_manager.py` (junto a save_logo_file).
- Actualizar HTML y borrar de legacy.py.

Módulo 4: Afiliados (El más grande)
- Crear `web/affiliates/__init__.py`, `auth_views.py`, `panel_views.py`, `tracking_views.py`.
- Mover rutas: `registro_afiliado`, `panel_afiliado`, `ver_producto`.
- Mover el `@app.before_request` de rastreo de cookies afiliadas a `tracking_views.py`.
- Actualizar HTML y borrar de legacy.py.

Futura Sesión Exclusiva: 
- Refactor de `web/imports/csv_views.py` para soportar 135,000 registros usando técnicas de bajo consumo de memoria.

En el archivo `estructura.txt` te paso el estado actual de la aplicacion.

Este es el legacy donde se esta haciendo el 'borrado' del proceso. Luego te paso los archivos de la app.py antigua.

```py
# web/legacy.py
from flask import Blueprint, redirect, url_for, flash, jsonify, request
from extensions import limiter

legacy_bp = Blueprint('legacy', __name__)

# --- RUTAS DE EXPORTACIÓN --- (Ya migradas a web/contacts/export_views.py)

# --- DIRECTORIO PÚBLICO --- (✅ MIGRADAS a web/directory/ - Borradas de aquí)

# --- PRODUCTOS ---
@legacy_bp.route("/negocio/<int:id>/nuevo-producto", methods=["POST"])
def agregar_producto(id):
    flash("⚠️ Módulo de productos en migración.", "info")
    return redirect(url_for('forms.editar_contacto', id=id))

# --- AFILIADOS ---
@legacy_bp.route("/registro-afiliado", methods=["GET", "POST"])
@limiter.limit("5 per minute")
def registro_afiliado():
    flash("⚠️ Módulo de afiliados en migración.", "info")
    return redirect(url_for('auth.login'))

@legacy_bp.route("/panel-afiliado")
def panel_afiliado():
    flash("⚠️ Panel de afiliados en migración.", "info")
    return redirect(url_for('contacts.index'))

@legacy_bp.route("/producto/<int:id>")
def ver_producto(id):
    return "Módulo de producto en migración", 404

# --- IMPORTACIONES --- (Ya migradas a web/imports/)
```