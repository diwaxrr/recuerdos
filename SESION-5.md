# 📝 ACTA - SESIÓN 5

**Estado: ✅ EXITOSA**

**Logros:**
1. ✅ Migración de `provincia`, `municipio`, `distrito_municipal`, `celular` en `contactos`.
2. ✅ Ingesta masiva de 135k registros csv, sin colapsar la RAM.
3. ✅ Asimilación inteligente (Cascading Match por Nombre y Teléfono).
4. ✅ Protección del OCR (Sus datos originales como `zona` no se sobrescribieron).
5. ✅ Limpieza de duplicados CSV.
6. ✅ **Resultado final: 147,697 negocios únicos en el Directorio.**

**Deuda técnica aceptada:** 

16,294 registros del OCR clásico sin ubicación nueva (por no existir en el CSV zona), pendientes de depuración manual.

**Siguiente parada (Sesión 6):** 

Sugerencias:

- Filtros de ubicación en la interfaz y estado de "Verificación" para que el equipo empiece a trabajar esos leads.
- Corregir detalles del Directorio y el index o CRM


# 🌊 Guía de Uso: El Río del Directorio (Ingesta de Datos)

Este documento explica cómo agregar nuevas fuentes de datos al Directorio, sin romper lo que ya existe.

## 🏗️ Arquitectura (Cómo funciona)

El sistema funciona en 3 fases:

1. **El Lago Central (`fuentes_crudas`):** Aquí caen los archivos crudos (CSV, JSON, Excel). Se guardan tal cual, en formato JSON, sin procesar. **No se altera la tabla `contactos` en esta fase.**
2. **El Río (Scripts de Procesamiento):** Leer el Lago, limpian los datos, buscan duplicados (Cascading Match) y llenan la tabla `contactos`.
3. **El Directorio (`contactos`):** La tabla final, limpia, lista para la web y el equipo de verificación.

---

## 🛠️ Paso a Paso para agregar NUEVOS datos

### Fase 1: Ingestar al Lago

*Usar interfaz web o el script `ingestor.py` para subir el archivo. Asegúrate de asignarle una etiqueta única en `fuente_origen` (ej: `PYMES_INFORMALES_2024`).*

### Fase 2: Preparar la DB (Si trae campos nuevos)
Si el nuevo CSV tiene campos que no existen en la DB (ej: `twitter`, `sector_economico`), primero debes crearlos:

```bash
# Ejemplo:
sqlite3 directorio.db "ALTER TABLE contactos ADD COLUMN twitter VARCHAR(100);"
```

### Fase 3: El Procesador (La Cascada)

**NUNCA intentes hacer UPDATE e INSERT al mismo tiempo, con lógica compleja desde Python.** 

El flujo probado y seguro que no pierde datos ni explota la RAM es:

1. **INSERTAR TODO lo nuevo:** 
   - Mete todos los registros del nuevo archivo directamente en `contactos` como "Nuevos Leads". 
   - No pienses en duplicados aún.
2. **ASIMILAR (El Match):**
   - Une los datos. 
   - Si el nuevo archivo trae un teléfono o dirección que al registro viejo le faltaba, pásaselo. 
   - Si el registro viejo ya tenía el dato y el nuevo está vacío, **NO LO SOBRESCRIBAS** (para esto sirve `COALESCE(NULLIF(campo_nuevo, ''), campo_viejo)`).
3. **LIMPIAR (Deduplicación):** 
   - Borra los registros sobrantes que quedaron después de la asimilación.

---

## ⚠️ Reglas de Oro

1. **Respeta el OCR/Histórico:** 
   - Los datos más viejos y difíciles de conseguir (como el OCR) son sagrados. 
   - La regla es: *El dato nuevo rellena los vacíos del dato viejo, pero nunca lo borra.*
2. **Cuidado con los TEXTOS LARGOS:** 
   - Los CSVs suelen traer direcciones de 500 caracteres. 
   - La DB tiene límites (ej: `VARCHAR(300)`). 
   - **Siempre usa `SUBSTR(campo, 1, limite)` al insertar** para evitar que SQLite trague el registro en silencio.
3. **Limpia los Teléfonos:** 
   - Antes de hacer match por teléfono, quítale guiones, espacios y paréntesis. 
   - Un "809-123-4567" no es igual a "8091234567" para la computadora.
4. **Modo Seguro en SQLite:** 
   - Siempre abre tus scripts de procesamiento masivo con `db.execute("PRAGMA journal_mode=WAL")`. 
   - Esto evita el error `database is locked` si alguien está navegando la web al mismo tiempo.
5. **Haz BACKUP antes de procesar:** 
   ```bash
   cp directorio.db directorio_backup_antes_de_procesar.db
   ```
---

## 📁 Archivos de referencia de la Sesión 5

- `core/utils/tel_sanitizer.py`: Limpia teléfonos.
- `core/utils/cascading_match.py`: Lógica de búsqueda (Nombre -> Tel -> Cel).
- `procesar_lago.py`: Orquestador de la Sesión 5.
- `add_campos_dgcp.py`: Ejemplo de cómo agregar campos nuevos.


## Estructura del proyecto

```
.
├── 7direct.service
├── add_campos_dgcp.py
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
│   └── __init__.py
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
├── directorio.db-shm
├── directorio.db-wal
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
- cada script debe tener menos de 150 lineas.

## Metodologia:

- Utilizamos el método **KISS**, (Keep It Simple and Smart)
- Solo damos un paso, cada vez
- Primero pensamos, analizamos, razonamos... luego codificamos

**Cuál es el siguiente paso que sugieres?**
**Qué codigo necesitas para el siguiente paso?**