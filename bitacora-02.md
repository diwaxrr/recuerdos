
# ERP 20Direct - Resumen del Proyecto

## Estado Actual (Sesión 1)

### 1. Base de Datos PostgreSQL
- **Base de datos:** `erp_v20` (puerto 5434)
- **Esquemas:**
  - `staging`: Datos crudos del CSV
  - `erp`: Datos normalizados (solo lectura para módulos)

#### Tablas creadas:
- `staging.pymes_raw`: 135,463 registros (datos crudos del CSV)
- `erp.empresas`: 135,463 registros (datos normalizados)
- `erp.contactos`: 126,198 registros
- `erp.provee`: 105,563 registros
- `erp.pymes`: Pymes/clientes del ERP
- `erp.modulos_disponibles`: Módulos disponibles
- `erp.pyme_modulos`: Módulos activos por pyme

#### Usuarios:
- `postgres`: Superusuario (solo para administración)
- `erp_reader`: Usuario de solo lectura para módulos (password: erp_reader_2024)

### 2. Microkernel del ERP
**Tecnología:** Node.js + Express.js
**Puerto:** 3000

#### Archivos creados en `/home/creamos/proyectos/20direct/microkernel/`:
1. `config.js` (30 líneas) - Configuración centralizada
2. `db.js` (50 líneas) - Conexión PostgreSQL (pool)
3. `loader.js` (100 líneas) - Cargador de módulos
4. `auth.js` (80 líneas) - Autenticación y sesiones
5. `router.js` (80 líneas) - Enrutador principal
6. `server.js` (100 líneas) - Servidor Express

#### Funcionalidad actual:
- ✅ Login/logout con sesiones
- ✅ Carga dinámica de módulos
- ✅ Verificación de permisos por módulo
- ✅ Conexión a DB de solo lectura
- ✅ Página de login funcional (public/index.html)

#### Pyme de prueba:
- Usuario: `valdes`
- Password: `valdes123`
- Módulos activos: CRM, Facturación

### 3. Estructura de Directorios

```
/home/creamos/proyectos/20direct/
├── microkernel/
│   ├── server.js          (servidor web, <100 líneas)
│   ├── router.js          (enrutador, <80 líneas)
│   ├── db.js              (conexión a PostgreSQL, <50 líneas)
│   ├── loader.js          (cargador de módulos, <100 líneas)
│   ├── auth.js            (autenticación básica, <80 líneas)
│   └── config.js          (configuración, <30 líneas)
├── modulos/              (módulos del ERP, vacío por ahora)
├── public/               (archivos estáticos)
│   ├── css/
│   │   └── estilos.css   (estilos del frontend)
│   ├── js/
│   │   └── login.js      (lógica del login)
│   └── index.html        (página de login)
├── node_modules/         (dependencias npm)
├── package.json          (configuración npm)
└── pymes.csv             (archivo original, 83MB)
```

### 4. Dependencias npm instaladas
- express
- express-session
- pg (node-postgres)
- bcrypt

## Próximos Pasos

### Panel de Control de la Base de Datos
**Objetivo:** Crear una interfaz web para administrar la base de datos

**Funcionalidades pendientes de definir:**
1. Acceso a datos crudos (staging) y normalizados (erp)
2. Gestión de pymes (crear, editar, activar/desactivar)
3. Gestión de módulos (asignar a pymes)
4. Búsqueda y filtrado de empresas
5. Limpieza de datos (normalizar formatos, eliminar espacios)
6. Estadísticas y reportes
7. Exportación de datos

**Decisiones pendientes:**
- ¿Quién usará el panel? (solo admin del sistema o también admins de pymes)
- ¿Qué permisos tendrá? (lectura/escritura en staging, lectura en erp)
- ¿Será parte del microkernel o aplicación separada?

## Reglas del Proyecto
1. **KISS** (Keep It Simple and Smart) - Un paso por interacción
2. **Modular**: modulos compuestos por plugins y plugins por scripts
3. **Cada script realiza, solamente, una tarea**
4. **Máximo 200 líneas por script**
5. **Idioma oficial: Español** (nombres de archivo, funciones, variables, constantes, objetos, clases...)
6. **DB central solo lectura para módulos**
7. **Para escribir, cada modulo lo hace en su propia db**
5. **Metodología:** Plan (análisis) → Build (ejecución paso a paso)

## Comandos Útiles

### Iniciar el microkernel:

```bash
cd /home/creamos/proyectos/20direct
node microkernel/server.js
```

### Conectarse a PostgreSQL:

```bash
sudo -u postgres psql -d erp_v20
```

### Conectarse como erp_reader:

```bash
psql -U erp_reader -d erp_v20 -h localhost -p 5434
```

## Notas Importantes
- El CSV original tiene 41 columnas, muchas con datos inconsistentes
- Algunas empresas tienen todos los campos vacíos excepto el nombre
- Los módulos futuros tendrán sus propias bases de datos para escritura
- El panel de control es el tercer componente del sistema