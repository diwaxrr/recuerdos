## 📝 NOTA SESION 6: KERNEL CRM DIRECTORIO BUSCADOR

Estás colaborando en el desarrollo de un sistema tecnológico para PyME's (Buscador, Directorio, Tienda, ERP, Asistentes Virtuales, etc.), construido sobre Flask (Python 3.13), y SQLite. El proyecto se maneja bajo la filosofía KISS (Keep It Simple and Smart), y desarrollado paso a paso para evitar el estrés, la sobrecarga y el caos operativo (KOS).

## 📍 ¿Qué se ha solucionado en la sesion 6, y dónde estamos?

   1. Módulo de Eliminación (web/contacts/delete_views.py): Se migró con éxito a SQLAlchemy (importando la db real desde extensions.py y los modelos desde la raíz models.py). Ahora funciona de forma aislada e independiente en el HTML gracias a un formulario oculto (style="display:none;") disparado por JavaScript desde el botón visual de eliminar. Esto solucionó los fallos de validación CSRF. El backend además limpia físicamente del disco duro (UPLOAD_FOLDER) el logo del contacto y las imágenes de sus productos asociados antes de ejecutar el DELETE.
   2. Módulo de Productos (web/products/form_views.py): Se blindó la creación de productos contra el error RuntimeError: The current Flask app is not registered with this 'SQLAlchemy' instance (causado por una inicialización duplicada en models.py). La solución limpia fue utilizar SQL nativo directo sobre la sesión unificada (db.session.execute(db.text(...))) importando db desde extensions.py.
   3. Validación del Límite de 10 Productos: El backend ahora cuenta los registros existentes con SQL antes de insertar. Si hay 10 o más, bloquea la acción. En el frontend (templates/editar.html), se usó Jinja2 para evaluar productos|length >= 10 y deshabilitar visualmente el botón de agregar, cambiándolo a color gris con un estilo en línea (inline CSS) y agregando el atributo disabled con el texto 🔒 Límite de productos alcanzado (Máx 10).
   4. Seguridad de Archivos: Al subir imágenes de productos, el backend utiliza secure_filename combinado con un prefijo único uuid.uuid4().hex[:8] para evitar la sobrescritura de archivos con nombres idénticos.

## 🔮 ¿Qué es lo que sigue? (temas sugeridos)

El usuario te compartirá el árbol de directorios actualizado (tree) y el código que consideres cuando abras la nueva sesión. Los objetivos en el horizonte son:

* Ruta de Edición/Eliminación Individual de Productos: Permitir que el usuario (pyme), gestione los productos existentes (cambiar precio, nombre o borrarlos uno por uno) manteniendo el control del almacenamiento físico.
* Hacer el Límite Dinámico: Preparar la arquitectura para que el tope de 10 productos no esté escrito en duro (hardcoded), sino que en el futuro se lea de una variable de configuración global (current_app.config) vinculada a un Panel de Control (Settings) o planes de afiliados.

## Estructura 

```
.
├── 7direct.service
├── add_campos_dgcp.py
├── app(Copia).py
├── app-copy-porsiacaso.py
├── app.py
├── backup_blueprints
│   ├── afiliados.py
│   ├── auth.py
│   ├── contactos.py
│   └── __init__.py
├── backup-db.sh
├── backups
│   └── directorio_20260720_103623.db
├── core
│   ├── database
│   │   ├── connection.py
│   │   ├── __init__.py
│   │   ├── migrations.py
│   │   └── __pycache__
│   ├── __init__.py
│   ├── __pycache__
│   └── utils
│       ├── cascading_match.py
│       ├── file_manager.py
│       ├── generators.py
│       ├── ingestor.py
│       ├── __init__.py
│       ├── parsers.py
│       ├── __pycache__
│       ├── sanitizers.py
│       └── tel_sanitizer.py
├── crm.log
├── directorio.db
├── directorio.db.backup
├── docs
│   └── directorio-copy-porsiacaso.db
├── extensions.py
├── fail2ban-7direct.conf
├── favicon.ico
├── img
├── instance
│   └── directorio.db
├── md2db.py
├── mezclar_duplicados.py
├── migrar_ocr.py
├── migrations
│   ├── alembic.ini
│   ├── env.py
│   ├── README
│   ├── script.py.mako
│   └── versions
│       └── eb64f44921a5_initial_migration_所有_modelos_sqlalchemy.py
├── models.py
├── nginx-7direct.conf
├── procesar_lago_original.py
├── procesar_lago.py
├── __pycache__
├── pytest.ini
├── rescatar_ocr.py
├── SEGURIDAD-VPS.sh
├── settings.py
├── static
│   ├── css
│   │   ├── base.css
│   │   ├── borrador.css
│   │   ├── componentes.css
│   │   ├── estilo2.css
│   │   ├── estilo-backup.css
│   │   ├── estilo.css
│   │   └── style.css
│   └── js
│       └── main.js
├── templates
│   ├── base.html
│   ├── buscador.html
│   ├── contacto.html
│   ├── directorio.html
│   ├── editar.html
│   ├── importar_md.html
│   ├── imports
│   │   └── importar_lago.html
│   ├── index.html
│   ├── login.html
│   ├── nuevo.html
│   ├── panel-afiliado.html
│   ├── reclamar.html
│   ├── registro-afiliado.html
│   ├── registro.html
│   └── sitio.html
├── tests
│   ├── conftest.py
│   ├── test_auth.py
│   ├── test_contactos.py
│   └── test_security.py
└── web
    ├── affiliates
    │   ├── auth_views.py
    │   ├── __init__.py
    │   ├── panel_views.py
    │   ├── __pycache__
    │   └── tracking_views.py
    ├── auth
    │   ├── __init__.py
    │   ├── __pycache__
    │   ├── routes.py
    │   └── session_helpers.py
    ├── buscador
    │   ├── __init__.py
    │   ├── __pycache__
    │   └── views.py
    ├── contacts
    │   ├── delete_views.py
    │   ├── detail_views.py
    │   ├── export_views.py
    │   ├── form_views.py
    │   ├── __init__.py
    │   ├── list_views.py
    │   ├── __pycache__
    │   └── tracking_views.py
    ├── directory
    │   ├── buscador_views.py
    │   ├── claim_views.py
    │   ├── __init__.py
    │   ├── public_views.py
    │   ├── __pycache__
    │   └── suggestion_views.py
    ├── imports
    │   ├── csv_views.py
    │   ├── __init__.py
    │   ├── lago_views.py
    │   ├── md_views.py
    │   └── __pycache__
    ├── __init__.py
    ├── legacy.py
    ├── products
    │   ├── form_views.py
    │   ├── __init__.py
    │   └── __pycache__
    └── __pycache__

37 directories, 103 files
```
# Proyecto

El proyecto busca crear una aplicación para PyME's, que pretende tener diferentes app; Directorio, Buscador, ERP (CRM, contabilidad, ventas, inventario, nómina...), Tienda, Catálogo... y cada app debería estar compuesta de Módulos. 

En el Directorio, que es público, los negocios pueden 'reclamar' su negocio, para editar su información, crear su propia Landing Page, y que pueda mostrar sus productos, bienes o servicios.

La idea es que cada Módulo sea independiente, que se conecte al y con el kernel.

También, se debe desarrollar un 'admin', para administrar la db y tener su propio CRM, que en el momento está en el `index`.


## Reglas:

- El sistema es modular
- Cada modulo esta compuesto de plugins
- Cada plugin esta compuesto de scripts
- Cada script hace solo y unicamente, UNA tarea
- Cada script debe tener menos de 150 lineas
- HTML, CSS Y JS deben ir separados

## Metodologia:

- Utilizamos el método **KISS**, (Keep It Simple and Smart)
- Solo damos un paso, cada vez
- Primero pensamos, analizamos, razonamos... luego codificamos

**Cuál es el siguiente paso que sugieres?**
**Qué codigo necesitas para el siguiente paso?**

