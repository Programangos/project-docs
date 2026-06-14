# skill_back — Revision de Buenas Practicas Backend SISA

## Descripcion

Revisa el codigo del backend de SISA (Django + DRF + PostgreSQL) y genera un reporte de cumplimiento de buenas practicas. Evalua convenciones de nombres, clean code, arquitectura en capas y principios SOLID/KISS. Solo reporta; al final pregunta si desea ayuda para corregir los problemas encontrados.

---

## Contexto del proyecto

**Stack:** Python 3.12+, Django, Django REST Framework, PostgreSQL (via Docker), SimpleJWT  
**Arquitectura:** capas separadas dentro de `src/core/`

```
src/core/
├── domain/        # Modelos Django (managed=False, mapean tablas de init.sql)
├── infra/         # Repositorios (unica capa que accede a la DB via ORM)
├── services/      # Logica de negocio (recibe repositorio por inyeccion)
├── controllers/   # Vistas DRF (APIView, solo HTTP, sin logica de negocio)
├── serializers/   # Serializadores DRF (validacion y conversion a JSON)
└── tests/         # Pruebas unitarias (pytest + unittest.mock)
```

**Dependencias entre capas (sentido permitido):**
```
controllers → services → infra → domain
serializers → domain
tests → services (mockeando infra)
```
Ninguna capa puede importar de una capa superior.

---

## Instrucciones de ejecucion

Ejecuta este skill sobre la carpeta `src/core/` del proyecto backend de SISA. Recorre los archivos Python de cada capa y genera el reporte completo usando el formato definido al final. No modifiques ningun archivo.

Para inspeccionar la estructura ejecuta:
```powershell
Get-ChildItem -Path src\core -Recurse -Filter *.py | Select-Object FullName
```

Para contar lineas de una funcion puedes leer el archivo directamente.

---

## Criterios de evaluacion

### SECCION 1 — Convenciones de nombres (Python)

| # | Prioridad | Criterio | Como verificarlo |
|---|-----------|----------|-----------------|
| 1.1 | MUST | Clases en PascalCase | Buscar `class ` y verificar formato |
| 1.2 | MUST | Funciones en snake_case | Buscar `def ` y verificar formato |
| 1.3 | MUST | Variables en snake_case | Revisar asignaciones dentro de funciones |
| 1.4 | MUST | Archivos en snake_case | Listar nombres de archivos .py |
| 1.5 | SHOULD | Nombres descriptivos (sin abreviaciones crípticas) | Revisar nombres de variables y parametros |
| 1.6 | SHOULD | Sin numeros magicos en el codigo | Buscar literales numericos sueltos (ej: 404, 201 hardcodeados sin constante) |

### SECCION 2 — Clean Code

| # | Prioridad | Criterio | Como verificarlo |
|---|-----------|----------|-----------------|
| 2.1 | MUST | Sin comentarios en el codigo (el codigo debe ser autoexplicativo) | Buscar `#` dentro de funciones y clases |
| 2.2 | MUST | Sin bloques try/except | Buscar `try:` y `except` en todo el codigo |
| 2.3 | MUST | Sin bloques switch (no existen en Python, pero si cadenas if/elif largas de mas de 3 ramas) | Buscar cadenas elif con mas de 3 ramas |
| 2.4 | MUST | Sin ifs o fors anidados (maximo 2 niveles de sangria dentro de una funcion) | Revisar profundidad de sangria en funciones |
| 2.5 | MUST | Funciones con maximo 3 argumentos | Contar parametros en cada `def` |
| 2.6 | MUST | Funciones de maximo 20 lineas | Contar lineas por funcion |
| 2.7 | MUST | Cada funcion hace una sola cosa (sin multiples niveles de abstraccion mezclados) | Revisar si hay funciones que hacen IO, logica y formato a la vez |
| 2.8 | MUST | Un bloque if/while contiene solo una llamada a funcion (sin logica compleja dentro) | Revisar bloques if/while |
| 2.9 | SHOULD | Funciones ordenadas: la que no depende de nada va arriba, la que depende va abajo | Revisar orden de funciones en cada archivo |
| 2.10 | SHOULD | Principio KISS (la solucion mas simple posible) | Revisar si hay sobre-ingenieria innecesaria |
| 2.11 | SHOULD | Principio SOLID: cada clase tiene una sola responsabilidad (SRP) | Verificar que controllers no tengan logica de negocio y services no hagan HTTP |
| 2.12 | SHOULD | DRY: sin bloques de codigo duplicados entre archivos del mismo tipo | Comparar implementaciones similares entre modulos |

### SECCION 3 — Arquitectura en capas

| # | Prioridad | Criterio | Como verificarlo |
|---|-----------|----------|-----------------|
| 3.1 | MUST | Todos los modelos en `domain/` tienen `managed = False` | Buscar `managed` en cada modelo |
| 3.2 | MUST | Ningun modelo en `domain/` tiene logica de negocio (solo campos y Meta) | Revisar que no haya metodos de negocio en los modelos |
| 3.3 | MUST | Solo `infra/` accede a `Model.objects.*` (ORM) | Buscar `.objects.` fuera de `infra/` |
| 3.4 | MUST | Los `services/` reciben el repositorio por inyeccion en `__init__` (no lo instancian directamente al inicio) | Verificar patron `def __init__(self, repository=None)` |
| 3.5 | MUST | Los `controllers/` no tienen logica de negocio ni consultas a DB | Revisar que controllers solo llamen al service y devuelvan Response |
| 3.6 | MUST | Los `controllers/` no importan nada de `infra/` ni de `domain/` directamente | Revisar imports en controllers |
| 3.7 | MUST | Los `serializers/` solo importan de `domain/` | Revisar imports en serializers |
| 3.8 | MUST | Los `tests/` usan MagicMock para reemplazar el repositorio (sin acceso real a DB) | Verificar uso de `MagicMock` y ausencia de `Model.objects` en tests |
| 3.9 | SHOULD | Los ForeignKey en `domain/` usan referencia en string `'core.Modelo'` (evita imports circulares) | Buscar `ForeignKey(` y verificar que el primer argumento sea string |
| 3.10 | SHOULD | Las tablas con PK compuesta tienen `unique_together` en el Meta | Verificar modelos de tablas like y vote |

### SECCION 4 — Documentacion y estilo

| # | Prioridad | Criterio | Como verificarlo |
|---|-----------|----------|-----------------|
| 4.1 | SHOULD | Los archivos `__init__.py` de cada carpeta estan presentes | Verificar existencia en domain, infra, services, controllers, serializers, tests |
| 4.2 | SHOULD | El archivo `pytest.ini` existe y tiene `DJANGO_SETTINGS_MODULE` configurado | Verificar existencia y contenido |
| 4.3 | SHOULD | El archivo `.env` existe en la raiz del backend y está listado en `.gitignore` | Verificar existencia del `.env` y que esté en `.gitignore` |
| 4.4 | SHOULD | `requirements.txt` incluye las dependencias de testing (pytest, pytest-django, coverage, flake8) | Leer requirements.txt |

---

## Formato de salida

Genera el reporte en este formato exacto:

```
# Reporte skill_back — SISA Backend
Fecha: <fecha actual>

## Seccion 1 — Convenciones de nombres
| # | Prioridad | Criterio | Estado | Recomendacion |
|---|-----------|----------|--------|---------------|
| 1.1 | MUST | Clases en PascalCase | ✅ | — |
| 1.2 | MUST | Funciones en snake_case | ✅ | — |
...

## Seccion 2 — Clean Code
| # | Prioridad | Criterio | Estado | Recomendacion |
...

## Seccion 3 — Arquitectura en capas
...

## Seccion 4 — Documentacion y estilo
...

## Resumen
- ✅ Cumple: X criterios
- ⚠️ Parcial: X criterios
- ❌ No cumple: X criterios

<Si hay problemas> ¿Deseas que te ayude a corregir los problemas encontrados?
<Si todo verde> ¡El proyecto cumple con todas las buenas practicas definidas!
```

**Leyenda de estados:**
- ✅ Cumple completamente
- ⚠️ Cumple parcialmente (indicar que falta en Recomendacion, maximo 10 palabras)
- ❌ No cumple (indicar que hacer en Recomendacion, maximo 10 palabras)

**Regla de Recomendacion:** solo aplica para ⚠️ y ❌. Para ✅ escribe `—`.
