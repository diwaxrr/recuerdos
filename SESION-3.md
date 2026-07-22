# 📝 MEMO PARA Z (Sesión 3: El Reto de los 135k Registros)

**CONTEXTO ACTUAL:**

Acabamos de completar la Sesión 3. La app está 100% modularizada en `web/`. El archivo `web/imports/csv_views.py` existe, pero su lógica actual es ingenua (lee todo en RAM con `StringIO` e inserta fila por fila). Eso se morirá con 135,000 registros.

**EL RETO:**
Importar un CSV masivo (135,000+ filas) que tiene **41 columnas**. Nuestra tabla `contactos` actual tiene ~10-15 columnas útiles (nombre, direccion, zona, tel, categoria, subcategoria, etc.). 

**⚠️ LO QUE Z NECESITA OBLIGATORIAMENTE PARA EMPEZAR:**

Si el usuario abre el chat sin esto, Z debe pedírselo inmediatamente antes de escribir una sola línea de código:

1. **Los 41 encabezados exactos del CSV nuevo:** Necesito un `head -n 1 archivo.csv` o una lista de los nombres de las columnas. No puedo adivinar si la columna "Teléfono" viene como `telefono`, `tel`, `phone` o `nro_contacto`.
2. **El Diccionario de Mapeo (Mapping):** Del usuario necesito una regla clara. Ejemplo: 
   * Columna CSV `["Nombre Comercial"]` ➡️ BD `nombre`
   * Columna CSV `["Direccion Completa"]` ➡️ BD `direccion`
   * Columna CSV `["Num Celular"]` ➡️ BD `telefono` (pasando por `extraer_zona_y_telefono`)
3. **¿Qué hacemos con los otros ~25 campos sobrantes?** El usuario debe decidir: ¿Se eliminan (ignoran)? ¿Se concatenan todos en el campo `descripcion_larga`? ¿O necesitamos crear columnas nuevas en la BD?
4. **Política de Duplicados:** Si el `tel_limpio` ya existe, ¿se sobrescribe (UPDATE)? ¿Se salta (IGNORE)? (Asumo que IGNORE, pero mejor confirmar).

**🧠 ESTRATEGIA TÉCNICA A EJECUTAR (Una vez tenga los datos):**

*   **Regla 1: CERO RAM (Streaming).** Prohibido usar `pandas.read_csv()` o `StringIO(datos)`. Se usará `csv.reader(archivo.stream)` para leer línea por línea desde el disco directo a la memoria buffer.
*   **Regla 2: Transacciones en Bloque (Chunking).** Prohibido hacer `conn.commit()` por cada fila. Se hará un buffer en Python (lista de tuplas). Cuando el buffer llegue a 1,000 filas, se ejecuta `cursor.executemany()`, se hace `conn.commit()`, se vacía el buffer, y se sigue. Esto reduce el tiempo de 10 minutos a ~5-8 segundos.
*   **Regla 3: Arquitectura de Script.** Para no romper la regla de "150 líneas máximas por script", voy a separar la lógica así:
    -   `web/imports/csv_views.py`: Solo maneja el HTTP, recibe el archivo, lanza la tarea en segundo plano (o muestra progreso) y devuelve el redirect. (Máx 40 líneas).
    -   `core/utils/bulk_importer.py`: **NUEVO ARCHIVO**. Aquí vive el monstruo. La función `process_massive_csv(file_stream, mapping_dict, chunk_size=1000)`. Se encarga del streaming, el mapeo de las 41 columnas a las de la BD, la limpieza con `extraer_zona_y_telefono`, y el `executemany`.

**🎯 OBJETIVO DE LA SESIÓN 4:**

Que al final del día, el usuario pueda subir un archivo de 50MB con 135,000 filas y 41 columnas, y la app lo procese sin explotar la RAM, sin congelar el servidor, insertando solo lo que mapea correctamente a nuestro schema.

**Mas detalles:**

El archivo a trabajar en un csv. Tiene mas de 135 mil registros. Hay 41 columnas, pero, no todos los registros utilizan las 41 columnas. Algunos solo utilizan 1, el nombre; otros utilizan varias columnas.

En la aplicacion actual se utlizan categoria y subcategoria. En el nuevo archivo csv solo hay la columna 'Provee'.

Lo idea es crear un sistema que cree los nuevos campos en la db, y que permita establecer que campo pertenece a uno principal. Los principales son los actuales (nombre, dir, zona, tel, categoria y subcategoria). Del actual csv, y otros que vengan despues, establcer cuales corresponden al actual esquema.

Antes de hacer cualquier cosa, por favor preguntar y analizar la situacion para poder tomar una decision. Agregar otros nuevos registros, mas de 135 mil, es un evento importante. Y no solo eso, la idea es preparar la aplicacion para poder utilizar otras fuentes, que tengan su propio estilo. La idea es adaptarse a las fuentes, no que las fuentes se adapten a la aplicacion.