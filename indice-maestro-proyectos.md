# Índice Maestro de Proyectos

> Actualizado: 2026-04-12  
> Fuente: `memoria.json` + historial de chats

---

## 📁 Proyectos

### 🔍 legal-rag
- **Estado**: En pausa (MVP validado)
- **Objetivo**: Sistema de consulta de leyes impositivas (330+ archivos .md)
- **Arquitectura**: Búsqueda híbrida (FTS SQLite + RAG semántico)
- **Unidad de información**: Artículo legal (no chunk fijo)
- **Keywords**: `legal`, `leyes`, `RAG`, `artículos`, `chunking`, `embeddings`, `índice`, `DeepSeek`
- **Archivos clave**: `/legal/chunker.py`, `/legal/index-leyes.md`
- **Resumen**: [2026-04-12.md](./2026-04-12.md)
- **Próximo paso**: Chunking inteligente para artículos >2k tokens

### 👥 crm-contactos
- **Estado**: Funcional con issues conocidos
- **Objetivo**: App web para red nacional de contactos con validación de cédula
- **Arquitectura**: Módulos reutilizables en carpeta `modules/`
- **Keywords**: `CRM`, `contactos`, `cédula`, `validación`, `exportación`, `cursor`
- **Archivos clave**: `/crm/validate_id.js`
- **Issues**: Cursor no se posiciona en "nombre", exportación limita a 100, estado visual de validación
- **Resumen**: [2026-04-12.md](./2026-04-12.md)

### 🐾 Vincent Pet Food
- **Estado**: En validación comercial
- **Objetivo**: Modelo de distribución sin inversión inicial en inventario
- **Enfoque**: Lealtad emocional con dueños de mascotas + validación social
- **Keywords**: `Vincent`, `Pet Food`, `proveedores`, `validación`, `logística`, `Europa`
- **Archivos clave**: Ninguno técnico aún
- **Próximo paso**: Definir flujo de pedidos y logística inicial

### ⚙️ wiki-comandos
- **Estado**: Concepto validado
- **Objetivo**: Herramienta de aprendizaje con búsqueda por sinónimos
- **Arquitectura**: Nanonúcleo + nanoplugins reutilizables
- **Keywords**: `wiki`, `comandos`, `programación`, `módulos`, `sinónimos`, `KISS`
- **Archivos clave**: `modules/` carpeta central
- **Próximo paso**: Estandarizar formato de módulos

### 🧠 contexto-memoria
- **Estado**: ✅ Protocolo validado (hoy)
- **Objetivo**: Recuperar contexto entre sesiones vía índice JSON público
- **Arquitectura**: `memoria.json` + archivos `.md` por fecha + GitHub Pages público
- **Keywords**: `contexto`, `memoria`, `índice`, `protocolo`, `web_extractor`, `history_retriever`
- **Archivos clave**: `memoria.json`, `2026-04-12.md`
- **Flujo**: Resumen sesión → índice JSON → búsqueda por keywords → reenganche

---

## 🔗 Enlaces rápidos
- [memoria.json](./memoria.json) - Índice de búsqueda
- [decision-log.md](./decision-log.md) - Registro de decisiones técnicas

---

## 📝 Notas
- Todos los resúmenes están en archivos con formato `YYYY-MM-DD.md`
- Para buscar: usa keywords de la columna correspondiente + `history_retriever`
- Para retomar: menciona el nombre del proyecto y consulta este índice