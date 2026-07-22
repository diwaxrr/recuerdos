# 📝 MEMO PARA IA (Sesión 5: El Río del Directorio y Cascading Match)


**CONTEXTO ACTUAL:**

La Sesión 4 fue un éxito. Tenemos un "Lago Central" (`fuentes_crudas`) funcionando. Ya ingerimos los datos viejos del OCR (etiqueta: `OCR_HISTORICO`) y el usuario ya subió el CSV de 135k registros (etiqueta: `CSV_DGCP_135K`) sin que explote la RAM. Todo está en JSON crudo.

**EL RETO DE LA SESIÓN 5:**

Construir el script `procesar_lago.py` que lea el Lago Central y pase los datos a la tabla real `contactos` (El Directorio). 

**⚠️ REGLAS DE NEGOCIO OBLIGATORIAS PARA ESTE SCRIPT:**

1.  **Cascading Match (El orden de prioridad para saber si es un registro existente o nuevo):**
    *   Intenta emparejar por **Nombre**. (Nota para IA: Si el nombre del CSV coincide exactamente con uno del OCR, se considera el mismo negocio).
    *   Si no hay match por nombre, intenta por **Teléfono Público** (`tel_limpio`).
    *   Si no hay match por teléfono, intenta por **Celular Público** (necesitaremos crear/mapear este campo nuevo si no existe en la tabla `contactos` actual).
2.  **Acción "Asimilar" (Si hay match en cualquiera de los 3 pasos anteriores):**
    *   El CSV es superior al OCR. Si el CSV tiene datos en esos campos, **SOBREESCRIBE** los datos del OCR en la tabla `contactos`.
3.  **Acción "Nuevo Lead" (Si NO hay match en ninguno de los 3 pasos):**
    *   Insertar como registro nuevo en `contactos`, **INCLUSO SI SOLO TIENE EL NOMBRE**. Quedarán campos vacíos (NULL), lo cual es correcto: son leads para que el equipo de verificación llame o investigue después.
4.  **Mapeo de Campos (Del JSON crudo a la tabla `contactos`):**
    *   `Nombre` -> `nombre`
    *   `Dirección` -> `direccion`
    *   `Teléfono` (del CSV) -> `telefono` / `tel_limpio`
    *   `Celular` (del CSV) -> `celular` (Revisar si existe en la DB actual, si no, agregarlo).
    *   `Municipio` / `Provincia` / `Dist. Municipal` -> Mapear a la lógica de `zona` y `ciudad` actual.
    *   `Provee` -> `categoria` (Temporalmente).

**OBJETIVO:**

Ejecutar el script, ver cómo se mezclan los datos actuales con los nuevos, respetando la cascada, y que la tabla `contactos` quede lista como un verdadero directorio con datos enriquecidos, y listos para la fase de verificación.

Estructura del proyecto
```
.
├── 7direct.service
├── app(Copia).py
├── app-copy-porsiacaso.py
├── app.py
├── backup-db.sh
├── backups
│   └── directorio_20260720_103623.db
├── blueprints
│   ├── afiliados.py
│   ├── auth.py
│   ├── contactos.py
│   ├── __init__.py
│   └── __pycache__
├── core
│   ├── database
│   │   ├── connection.py
│   │   ├── __init__.py
│   │   ├── migrations.py
│   │   └── __pycache__
│   ├── __init__.py
│   ├── __pycache__
│   └── utils
│       ├── file_manager.py
│       ├── generators.py
│       ├── ingestor.py
│       ├── __init__.py
│       ├── parsers.py
│       ├── __pycache__
│       └── sanitizers.py
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
├── __pycache__
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
│   ├── imports
│   │   ├── importar_lago.html
│   │   └── importar_lago.html -->
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
    ├── contacts
    │   ├── detail_views.py
    │   ├── export_views.py
    │   ├── form_views.py
    │   ├── __init__.py
    │   ├── list_views.py
    │   ├── __pycache__
    │   └── tracking_views.py
    ├── directory
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
```

## Reglas:

- El sistema es modular
- Cada modulo esta compuesto de plugins
- Cada plugin esta compuesto de scripts
- Cada script hace solo y unicamente, UNA tarea
- cada script debe tener menos de 15 lineas.

## Metodologia:

**KISS (Keep It Simple and Smart)**

- Solo un paso a la vez
- Primero razonar, luego actuar

Que info necesitas para el siguiente paso?