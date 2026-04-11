# Decision Log - Registro de Decisiones Técnicas

> Actualizado: 2026-04-12  
> Propósito: Evitar reinventar soluciones y mantener coherencia entre sesiones

---

## 🗓️ 2026-04-12 - Protocolo de Memoria

| Decisión | Alternativas consideradas | Razón | Impacto |
|----------|--------------------------|-------|---------|
| Usar índice JSON público + keywords para recuperación de contexto | NotebookLM API, Notion, DB privada | Simpleza extrema: sin auth, sin configuración, universal | ✅ Yo puedo leer vía `web_extractor`; tú mantienes control total |
| Archivos de resumen con formato `YYYY-MM-DD.md` | Nombres descriptivos, carpetas por proyecto | La fecha ordena cronológicamente sin ambigüedad | ✅ Búsqueda manual y automática más predecible |
| Keywords como "puente" entre chats y memoria | Tags complejos, categorías jerárquicas | KISS: palabras simples que tú ya usas al hablar | ✅ `history_retriever` funciona mejor con lenguaje natural |
| Seguridad por oscuridad (datos crípticos sin contexto) | Encriptación, auth, tokens | El valor está en el contexto, no en el dato aislado | ✅ Suficiente para nuestro caso; cero fricción de configuración |

---

## 🗓️ 2026-04-01 - Legal-RAG

| Decisión | Alternativas consideradas | Razón | Impacto |
|----------|--------------------------|-------|---------|
| Unidad de información = Artículo legal | Chunk fijo (512 tokens), párrafo | El Artículo es la unidad legal real; preserva significado | ✅ Precisión en respuestas; ❌ Requiere chunking inteligente para artículos largos |
| Búsqueda híbrida: FTS SQLite + RAG semántico | Solo RAG, solo FTS, vectores puros | FTS para números de ley exactos ("11-92"), RAG para intención | ✅ Mejor precisión combinada; ❌ Mayor complejidad de implementación |
| Formato de entrada: Markdown con encabezados anidados | JSON estructurado, XML, HTML | `.md` es legible, editable y preserva jerarquía con `#` | ✅ Fácil de mantener; ❌ Requiere parseo robusto de anidamiento |
| API de IA: DeepSeek prioritaria | Qwen, GPT-4, LLM local | Mejor relación calidad/precio; Qwen como fallback | ✅ Costos controlados en fase demo; ❌ Dependencia externa |

---

## 🗓️ 2026-03-15 - Wiki Comandos / Módulos

| Decisión | Alternativas consideradas | Razón | Impacto |
|----------|--------------------------|-------|---------|
| Arquitectura: nanonúcleo + nanoplugins | Monolito modular, microservicios completos | KISS: solo lo necesario, sin sobrecarga inicial | ✅ Rápido de iterar; ❌ Requiere disciplina para no "ensuciar" el núcleo |
| Búsqueda por sinónimos | Búsqueda exacta, embeddings semánticos | Los programadores buscan con lenguaje natural ("cómo leer archivo") | ✅ Más accesible para no expertos; ❌ Requiere mantenimiento de diccionario de sinónimos |
| Carpeta `modules/` centralizada | Módulos dispersos por proyecto | Reutilización real requiere punto único de verdad | ✅ Evita duplicación; ❌ Riesgo de acoplamiento si no se documenta bien |

---

## 🗓️ 2026-03-04 - CRM Contactos

| Decisión | Alternativas consideradas | Razón | Impacto |
|----------|--------------------------|-------|---------|
| Validación de cédula como módulo independiente | Validación inline, validación en backend | Separar lógica compleja facilita testing y reuso | ✅ Módulo reusable para otros proyectos; ❌ Requiere interfaz clara de entrada/salida |
| Arquitectura modular por carpetas | Todo en un archivo, MVC tradicional | KISS + reutilización: copiar carpeta base + plugins | ✅ Rápido de adaptar; ❌ Requiere convención estricta de nombres |
| Cursor se posiciona en "nombre" al crear contacto | Cursor en primer campo válido, sin posicionamiento | UX mínima: reducir fricción en entrada de datos | ✅ Mejor experiencia; ❌ Detalle fácil de olvidar en refactor |

---

## 🗓️ 2026-03-03 - Vincent Pet Food

| Decisión | Alternativas consideradas | Razón | Impacto |
|----------|--------------------------|-------|---------|
| Modelo: proveedor aporta producto, tú validas mercado | Dropshipping clásico, compra de inventario inicial | Validar demanda sin riesgo financiero | ✅ Seguro para ti; ❌ Requiere confianza mutua con proveedor |
| Enfoque en lealtad emocional con dueños | Enfoque en precio, en características técnicas | Las mascotas generan conexión afectiva, no racional | ✅ Diferenciación real; ❌ Requiere lenguaje específico por audiencia |
| Validación local antes de escalar | Lanzamiento directo multi-ciudad | Aprender rápido con bajo costo | ✅ Iteración ágil; ❌ Crecimiento más lento inicialmente |

---

## 🔄 Patrones transversales (aprendizajes recurrentes)

1. **KISS primero**: Siempre empezar con lo mínimo viable, añadir complejidad solo si se valida
2. **Módulos reutilizables**: Cada solución debe poder extraerse como "plugin" para otros proyectos
3. **Texto plano > Binario**: `.md`, `.json`, `.txt` son más portables y fáciles de debuggear
4. **HTTP público > Auth complejo**: Para nuestro caso, la simplicidad de acceso vale más que la seguridad formal
5. **Keywords naturales**: Usar el lenguaje que tú usas al hablar, no tags técnicos inventados

---

## 📝 Cómo usar este archivo

1. **Antes de empezar**: Revisa si hay decisiones previas sobre el tema
2. **Al tomar una decisión nueva**: Añádela al final con fecha, alternativas y razón
3. **Al reenganchar**: Busca por palabra clave en la columna "Decisión" o "Keywords"
4. **Para auditar**: Revisa "Impacto" para entender trade-offs ya aceptados