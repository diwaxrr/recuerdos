# README 00Direct ERP PyME's

**Objetivo**

00Direct es un ERP para pymes, de libre acceso y desarrollo para sus miembros.

**Metodología**

Aplica la metodología KISS (Keep It Simple and Smart).

**Diseño**

Su desarrollo es colaborativo; cualquiera puede desarrollar módulos, plugins o scripts, para su uso o el de la comunidad.

Su desarrollo es modular; el ERP está compuesto de módulos, los módulos de plugins, los plugins de scripts.

Cada script realiza sólo una tarea.

Ningún script puede tener más de 150 líneas de código, salvo casos excepcionales.

El idioma oficial de 0Direct ERP para PyME's es el Español.

Debe haber una DB central, de sólo lectura para los módulos.

La DB central debe tener su propio Panel de Control.

El Panel de Control no sólo administra la DB central, también los módulos y el ERP en general.

Para la escritura, cada módulo tiene su propia db.

La tecnología utilizada es; Postgres, SQLite, Node, HTML, CSS, JS (en archivos separados), y Python.

Debe haber una plantilla, para que el ERP tenga el mismo estilo.
