# skill_front — Revision de Buenas Practicas Frontend SISA

## Descripcion

Revisa el codigo del frontend de SISA (Vue 3 + TypeScript + Pinia + Tailwind + Axios) y genera un reporte de cumplimiento de buenas practicas. Evalua convenciones de nombres, clean code, arquitectura en capas y separacion de responsabilidades. Solo reporta; al final pregunta si desea ayuda para corregir los problemas encontrados.

---

## Contexto del proyecto

**Stack:** Vue 3, TypeScript, Pinia, Vue Router, Axios, Tailwind CSS, Lucide Vue  
**Arquitectura:** capas separadas dentro de `src/`

```
src/
├── components/    # Componentes UI reutilizables (SisaHeader.vue, SisaFooter.vue)
├── views/         # Vistas por pantalla (orquestan componentes, conectan al store)
├── pages/         # Paginas compuestas (agrupan vistas con layout)
├── router/        # Configuracion de rutas y guardias de navegacion (index.js)
├── services/      # Llamadas HTTP a la API (apiClient.js, authService.js)
├── stores/        # Estado global con Pinia (authStore.js)
└── types/         # Interfaces y tipos TypeScript + constantes (index.ts)
```

**Dependencias entre capas (sentido permitido):**
```
views        → stores, (types)
components   → (types)
stores       → services
services     → apiClient
router       → stores
types        → (nada)
```
Ninguna capa puede importar de una capa superior. Las vistas no llaman directamente a services, lo hacen via el store.

---

## Instrucciones de ejecucion

Ejecuta este skill sobre la carpeta `src/` del proyecto frontend de SISA. Recorre los archivos `.vue`, `.ts` y `.js` de cada capa y genera el reporte completo usando el formato definido al final. No modifiques ningun archivo.

Para inspeccionar la estructura ejecuta:
```powershell
Get-ChildItem -Path src -Recurse -Include *.vue,*.ts,*.js | Select-Object FullName
```

Para leer un archivo especifico:
```powershell
Get-Content src\stores\authStore.js
```

---

## Criterios de evaluacion

### SECCION 1 — Convenciones de nombres (Vue + TypeScript)

| # | Prioridad | Criterio | Como verificarlo |
|---|-----------|----------|-----------------|
| 1.1 | MUST | Componentes y vistas en PascalCase (`RegisterForm.vue`, `SisaHeader.vue`) | Listar archivos .vue y verificar formato |
| 1.2 | MUST | Servicios y stores en camelCase (`authService.js`, `authStore.js`) | Listar archivos en services/ y stores/ |
| 1.3 | MUST | Interfaces TypeScript en PascalCase (`RegistrationForm`, `CarreraOption`) | Revisar types/index.ts |
| 1.4 | MUST | Variables y funciones en camelCase (`handleSubmit`, `isAuthenticated`) | Revisar interior de .vue y .ts |
| 1.5 | MUST | Stores de Pinia nombradas con prefijo `use` y sufijo `Store` (`useAuthStore`) | Verificar `defineStore` en cada store |
| 1.6 | SHOULD | Nombres descriptivos sin abreviaciones crípticas | Revisar nombres de variables y props |
| 1.7 | SHOULD | Sin numeros magicos en el codigo (ej: `status === 401` sin constante) | Buscar literales numericos sueltos |

### SECCION 2 — Clean Code (JavaScript / TypeScript)

| # | Prioridad | Criterio | Como verificarlo |
|---|-----------|----------|-----------------|
| 2.1 | MUST | Sin comentarios en el codigo (el codigo debe ser autoexplicativo) | Buscar `//` y `/* */` dentro de funciones y `<script>` |
| 2.2 | MUST | Sin bloques try/catch en los stores o services | Buscar `try {` y `catch` en stores/ y services/ |
| 2.3 | MUST | Sin ifs o ternarios anidados de mas de 2 niveles | Revisar profundidad de condiciones en funciones |
| 2.4 | MUST | Funciones con maximo 3 argumentos | Contar parametros en cada funcion |
| 2.5 | MUST | Funciones de maximo 20 lineas | Contar lineas por funcion en script setup y stores |
| 2.6 | MUST | Cada funcion hace una sola cosa | Verificar que funciones de validacion no tambien hagan submit, etc. |
| 2.7 | MUST | Sin JSDoc (`/** */`) en el codigo | Buscar bloques `/**` en services y stores |
| 2.8 | SHOULD | Principio KISS: sin sobre-ingenieria para casos simples | Revisar si hay abstracciones innecesarias |
| 2.9 | SHOULD | DRY: sin logica de validacion duplicada entre vistas | Comparar funciones validateForm entre componentes |
| 2.10 | SHOULD | Principio SRP: cada archivo tiene una sola responsabilidad | Verificar que stores no hagan llamadas HTTP directas y services no manejen estado |

### SECCION 3 — Arquitectura en capas Vue

| # | Prioridad | Criterio | Como verificarlo |
|---|-----------|----------|-----------------|
| 3.1 | MUST | Las vistas (`views/`) no importan directamente de `services/` | Buscar imports de services dentro de views/ |
| 3.2 | MUST | Solo los stores importan de `services/` | Buscar imports desde `services/` en views/, components/ y pages/; solo deben aparecer en stores/ |
| 3.3 | MUST | Los stores usan `defineStore` de Pinia (no estado global manual) | Verificar uso de `defineStore` en cada archivo de stores/ |
| 3.4 | SHOULD | Si existen rutas protegidas, se implementan mediante guardias de navegacion centralizadas (`beforeEach`) | Verificar existencia de `router.beforeEach` en router/index.js si hay rutas con meta de autenticacion |
| 3.5 | MUST | Las llamadas HTTP utilizan un cliente centralizado compartido. No existen configuraciones duplicadas de Axios o fetch | Verificar que no haya llamadas `axios.create` ni `fetch` con baseURL fuera del cliente centralizado |
| 3.6 | MUST | Los tipos e interfaces compartidos entre multiples archivos estan en `types/`. Se permiten tipos locales definidos inline cuando solo se usan dentro del archivo | Buscar `interface` o `type` fuera de types/ y verificar si se reutilizan en otros archivos |
| 3.7 | MUST | Las constantes reutilizadas en multiples archivos estan centralizadas en `types/` o `constants/`. Se permiten constantes locales cuando solo se usan dentro del archivo | Buscar arrays u objetos de constantes definidos dentro de archivos .vue o stores y verificar si se repiten en otros archivos |
| 3.8 | SHOULD | Los componentes en `components/` no contienen logica de negocio ni acceden directamente a stores de dominio | Revisar imports y logica dentro de cada archivo en components/ |
| 3.9 | SHOULD | El cliente HTTP centralizado tiene interceptor de respuesta que maneja errores de autenticacion (401) | Verificar `interceptors.response` en el cliente HTTP |
| 3.10 | SHOULD | El almacenamiento local (`localStorage`, `sessionStorage`) se usa solo dentro de stores/, no en vistas ni servicios | Buscar `localStorage` y `sessionStorage` fuera de stores/ |

### SECCION 4 — TypeScript y tipado

| # | Prioridad | Criterio | Como verificarlo |
|---|-----------|----------|-----------------|
| 4.1 | MUST | Los componentes `.vue` usan `<script setup lang="ts">` | Verificar atributo lang en cada script setup |
| 4.2 | MUST | Las props de componentes tienen tipos definidos con `defineProps<{}>()` | Verificar uso de genericos en defineProps |
| 4.3 | MUST | Los emits tienen tipos definidos con `defineEmits<{}>()` | Verificar uso de genericos en defineEmits |
| 4.4 | SHOULD | Las variables reactivas tienen tipo explicito cuando no es inferible (`ref<string>('')`) | Revisar refs en script setup |
| 4.5 | SHOULD | No se usa `any` como tipo en ningun archivo | Buscar `: any` en archivos .ts y .vue |

### SECCION 5 — Estructura y configuracion del proyecto

| # | Prioridad | Criterio | Como verificarlo |
|---|-----------|----------|-----------------|
| 5.1 | MUST | Existe `vite.config.ts` en la raiz | Verificar existencia |
| 5.2 | MUST | Existe `tsconfig.json` en la raiz | Verificar existencia |
| 5.3 | MUST | Tailwind CSS esta correctamente configurado para la version utilizada (v3: `tailwind.config.js` en raiz; v4: directiva `@import "tailwindcss"` en el CSS principal) | Verificar segun version: existencia de `tailwind.config.js` o directiva en CSS |
| 5.4 | MUST | `package.json` incluye las dependencias principales: vue, pinia, vue-router, axios, lucide-vue-next | Leer package.json y verificar dependencies |
| 5.5 | SHOULD | Existe `.env.example` o `.env` con `VITE_API_URL` o similar para la URL del backend | Verificar existencia |
| 5.6 | SHOULD | El `baseURL` del cliente HTTP usa variable de entorno (`import.meta.env.VITE_*`) en lugar de estar hardcodeado | Verificar el cliente HTTP centralizado en services/ |

---

## Formato de salida

Genera el reporte en este formato exacto:

```
# Reporte skill_front — SISA Frontend
Fecha: <fecha actual>

## Seccion 1 — Convenciones de nombres
| # | Prioridad | Criterio | Estado | Recomendacion |
|---|-----------|----------|--------|---------------|
| 1.1 | MUST | Componentes en PascalCase | ✅ | — |
...

## Seccion 2 — Clean Code
| # | Prioridad | Criterio | Estado | Recomendacion |
...

## Seccion 3 — Arquitectura en capas Vue
...

## Seccion 4 — TypeScript y tipado
...

## Seccion 5 — Estructura y configuracion
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
