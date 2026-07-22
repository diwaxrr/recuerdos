
# SESION GUARDADA - REFACTORING FLASK KERNEL - 18 JULIO 20:45

Contexto: Estoy refactorizando un archivo monolítico app.py (~2000 líneas) de un CRM/Directorio Flask, transformándolo en una arquitectura modular tipo "Kernel".
Ya superamos el error de Import Circular y el sistema arranca perfectamente.

REGLAS DE ORO DEL PROYECTO (No romper estas bajo ninguna circunstancia):
1. Jerarquía: Módulo -> Plugin -> Script.
2. Cada Script hace UNA SOLA tarea.
3. Ningún Script puede pasar de 150 líneas.
4. CERO importaciones desde app.py. Si un script necesita la DB, importa de `core.database.connection`. Si necesita el limitador, importa de `extensions`.

ESTADO ACTUAL (Lo que ya está funcionando):
✅ Kernel (app.py): Crea la app y registra blueprints.
✅ Configuración (settings.py y extensions.py): Aisladas.
✅ Core Database (core/database/): Conexión y migraciones SQL.
✅ Core Utils (core/utils/): Sanitizadores, parsers, file_manager, generators.
✅ Plugin Auth (web/auth/): Login, registro, logout, perfil.
✅ Plugin Contacts (web/contacts/): Index, ver detalle, crear, editar, seguimientos.
✅ Plugin Legacy (web/legacy.py): Un blueprint "puente" que captura todas las rutas que aún no se han migrado para evitar errores 404/500. Al completar una migración, se debe eliminar la ruta correspondiente de legacy.py.

LO QUE QUEDA POR HACER (Las tareas pendientes sacadas de legacy.py):
El objetivo ahora es ir sacando las funciones del archivo viejo (app_old.py), creando sus propios scripts modulares y actualizando los url_for() en los templates HTML. El orden sugerido es:

1. Módulo Exportar/Importar: Crear `web/contacts/export_views.py` (CSV/JSON) y `web/imports/` (CSV y Markdown).
2. Módulo Directorio: Crear `web/directory/` (Público y buscador de sugerencias JSON).
3. Módulo Productos: Crear `web/products/` (Agregar producto a contacto).
4. Módulo Afiliados: Crear `web/affiliates/` (Registro afiliado, panel, reclamar negocio, rastreo de cookies).

Aquí está el árbol actual de mi proyecto para que veas dónde estamos:
[PEGA AQUÍ EL COMANDO tree -I '__pycache__|*.pyc|*.db|migrations|instance|img|static|docs|tests|node_modules')

```
.
├── 7direct.service
├── app(Copia).py
├── app.py
├── backup-db.sh
├── blueprints
│   ├── afiliados.py
│   ├── auth.py
│   ├── contactos.py
│   ├── __init__.py
│   └── __pycache__
│       ├── afiliados.cpython-313.pyc
│       ├── auth.cpython-313.pyc
│       ├── contactos.cpython-313.pyc
│       └── __init__.cpython-313.pyc
├── core
│   ├── database
│   │   ├── connection.py
│   │   ├── __init__.py
│   │   ├── migrations.py
│   │   └── __pycache__
│   │       ├── connection.cpython-313.pyc
│   │       ├── __init__.cpython-313.pyc
│   │       └── migrations.cpython-313.pyc
│   ├── __init__.py
│   ├── __pycache__
│   │   └── __init__.cpython-313.pyc
│   └── utils
│       ├── file_manager.py
│       ├── generators.py
│       ├── __init__.py
│       ├── parsers.py
│       ├── __pycache__
│       │   ├── file_manager.cpython-313.pyc
│       │   ├── generators.cpython-313.pyc
│       │   ├── __init__.cpython-313.pyc
│       │   └── sanitizers.cpython-313.pyc
│       └── sanitizers.py
├── crm.log
├── directorio.db
├── docs
│   └── directorio-copy-porsiacaso.db
├── extensions.py
├── fail2ban-7direct.conf
├── favicon.ico
├── img
├── instance
│   └── directorio.db
├── md2db.py
├── migrations
│   ├── alembic.ini
│   ├── env.py
│   ├── README
│   ├── script.py.mako
│   └── versions
│       └── eb64f44921a5_initial_migration_所有_modelos_sqlalchemy.py
├── models.py
├── nginx-7direct.conf
├── __pycache__
│   ├── app.cpython-313.pyc
│   ├── config.cpython-313.pyc
│   ├── extensions.cpython-313.pyc
│   ├── models.cpython-313.pyc
│   └── settings.cpython-313.pyc
├── pytest.ini
├── SEGURIDAD-VPS.sh
├── settings.py
├── static
│   ├── css
│   │   ├── borrador.css
│   │   ├── estilo2.css
│   │   ├── estilo-backup.css
│   │   ├── estilo.css
│   │   └── style.css
│   └── js
│       └── main.js
├── templates
│   ├── base.html
│   ├── contacto.html
│   ├── directorio.html
│   ├── editar.html
│   ├── importar_md.html
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
    ├── auth
    │   ├── __init__.py
    │   ├── __pycache__
    │   │   ├── __init__.cpython-313.pyc
    │   │   ├── routes.cpython-313.pyc
    │   │   └── session_helpers.cpython-313.pyc
    │   ├── routes.py
    │   └── session_helpers.py
    ├── contacts
    │   ├── detail_views.py
    │   ├── form_views.py
    │   ├── __init__.py
    │   ├── list_views.py
    │   ├── __pycache__
    │   │   ├── detail_views.cpython-313.pyc
    │   │   ├── form_views.cpython-313.pyc
    │   │   ├── __init__.cpython-313.pyc
    │   │   ├── list_views.cpython-313.pyc
    │   │   └── tracking_views.cpython-313.pyc
    │   └── tracking_views.py
    ├── __init__.py
    ├── legacy.py
    └── __pycache__
        ├── __init__.cpython-313.pyc
        └── legacy.cpython-313.pyc
``` 
Fin Sesion guardada
