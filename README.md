# 🧪 PokeAPI Testing Suite - Postman & Automated Scripts

Aseguramiento de calidad y análisis técnico sobre la REST API pública **PokeAPI** (v2). El proyecto consta del diseño, ejecución y análisis de un set de pruebas (5 positivas y 5 negativas) para evaluar la resiliencia del backend, la consistencia de sus respuestas y la experiencia de integración desde la perspectiva del cliente.

---

## 🎯 Objetivo del Proyecto
Aplicar pruebas automatizadas y manuales sobre servicios REST públicos para consolidar buenas prácticas de testing de API, analizando códigos de estado HTTP, estructuras de carga útil (Payload JSON), tiempos de respuesta (Performance) y aserciones mediante scripts en Postman.

## 🎛️ Alcance Técnico
*   **API Base URL:** `https://pokeapi.co/api/v2/` (Manejada dinámicamente mediante la variable de colección `{{base_url}}`).
*   **Casos de Prueba:** 10 Requests estructuradas (5 Caminos Felices / 5 Escenarios Negativos o de Borde).
*   **Automatización:** Inclusión de scripts de validación con JavaScript (`pm.test`) en las pestañas de *Tests* para ejecuciones automatizadas mediante el *Collection Runner*.

---

## 📊 Cuadro de Mando y Resumen de Resultados

El ciclo de ejecución arrojó una tasa de éxito esperada del **70%**. Las 3 desviaciones detectadas (*Fail*) representan discrepancias entre las especificaciones teóricas del estándar REST y las decisiones de arquitectura del framework del servicio (Express.js), constituyendo hallazgos críticos de diseño.

| ID | Caso de Prueba | Método | Resultado Esperado | Resultado Real | Estado | Categoría |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **API-01** | Obtener Pokémon existente | `GET` | 200 OK | 200 OK | ✅ Pass | Funcional (Positivo) |
| **API-02** | Obtener Pokémon por ID | `GET` | 200 OK | 200 OK | ✅ Pass | Funcional (Positivo) |
| **API-03** | Listado paginado (Query params) | `GET` | 200 OK | 200 OK | ✅ Pass | Lógica de Negocio |
| **API-04** | Consultar relación por Tipo | `GET` | 200 OK | 200 OK | ✅ Pass | Integridad de Datos |
| **API-05** | Tiempo de respuesta y Headers | `GET` | < 1000ms | 124ms | ✅ Pass | Performance / Estándar |
| **API-06** | ID fuera de rango numérico | `GET` | 404 Not Found | 404 Not Found | ✅ Pass | Manejo de Errores |
| **API-07** | Nombre de recurso mal escrito | `GET` | 404 Not Found | 400 Bad Request | ❌ Fail | Hallazgo de Enrutamiento |
| **API-08** | Path de endpoint inexistente | `GET` | 404 Not Found | 400 Bad Request | ❌ Fail | Hallazgo de Enrutamiento |
| **API-09** | Método HTTP no soportado | `POST` | 405 Method Not Allowed | 404 Not Found | ❌ Fail | Arquitectura de Router |
| **API-10** | Parámetro de paginación inválido | `GET` | 400 Bad Request | 200 OK | ✅ Pass | Permisividad de Query |

---

## 🔍 Análisis Detallado de Ejecución

### 🟢 Escenarios Positivos

#### API-01 — Obtener un Pokémon existente por Nombre
*   **Endpoint:** `GET {{base_url}}pokemon/pikachu`
*   **Métricas:** 200 OK | ~458ms
*   **Validación:** Verifica que la carga útil retorne las propiedades clave del esquema (ej. `abilities`). Comportamiento correcto para recursos indexados.

#### API-02 — Obtener un Pokémon por ID Numérico
*   **Endpoint:** `GET {{base_url}}pokemon/6`
*   **Métricas:** 200 OK | ~323ms
*   **Validación:** Confirma la flexibilidad del backend para resolver dinámicamente tanto el identificador alfanumérico (`string`) como el numérico (`integer`), apuntando al mismo recurso (Charizard).

#### API-03 — Listado Paginado de Recursos
*   **Endpoint:** `GET {{base_url}}pokemon?limit=20&offset=0`
*   **Métricas:** 200 OK | ~114ms
*   **Validación:** El payload cumple las directrices estables de paginación del lado del servidor suministrando llaves explícitas de control: `count`, `next`, `previous` y el arreglo de `results` limitado exactamente a 20 entradas.

#### API-04 — Consultar Relaciones de Recursos (Tipo)
*   **Endpoint:** `GET {{base_url}}type/electric`
*   **Métricas:** 200 OK | ~164ms
*   **Validación:** Evalúa la consistencia relacional de la base de datos (Tipo ➡️ Lista de Pokémon asociados).

#### API-05 — Acuerdo de Nivel de Servicio (SLA) & Headers
*   **Endpoint:** `GET {{base_url}}ability/overgrow`
*   **Métricas:** 200 OK | ~124ms
*   **Validación:** Asegura que los tiempos de respuesta de transacciones simples permanezcan bajo el umbral de aceptación estándar de < 1000ms y valida que las cabeceras entreguen el formato correcto `Content-Type: application/json`.

---

### 🔴 Escenarios Negativos y Hallazgos Clave de QA

#### API-06 — Recurso Inexistente (ID Fuera de Rango)
*   **Endpoint:** `GET {{base_url}}pokemon/999999`
*   **Métricas:** 404 Not Found | ~1647ms
*   **Análisis:** Comportamiento alineado a la especificación REST. No obstante, el tiempo de respuesta elevado evidencia un impacto en la performance provocado por una estrategia de fallos en caché (`cf-cache-status: MISS`).

#### API-07 y API-08 — Nombres de Recursos y Paths Mal Escritos
*   **Endpoints:** `GET {{base_url}}pikachuu` / `GET {{base_url}}pokemonn/1`
*   **Métricas:** 400 Bad Request | ~261ms / ~452ms
*   **🚨 Hallazgo:** A diferencia de la convención común en APIs REST de retornar un error genérico `404 Not Found` ante cualquier elemento no encontrado, la arquitectura interna de PokeAPI discrimina rigurosamente a nivel del Router. Si el string de la URI no logra coincidir con los patrones expresos de sus rutas declaradas, el backend lo intercepta como una petición mal estructurada (`400 Bad Request`).

#### API-09 — Invocación con Método HTTP No Soportado
*   **Endpoint:** `POST {{base_url}}pokemon/pikachu`
*   **Métricas:** 404 Not Found | ~219ms
*   **🚨 Hallazgo:** Enviar un verbo no registrado (POST) a un endpoint estrictamente de lectura (GET) provoca una respuesta por defecto del framework base (Express.js): `Cannot POST /api/v2/pokemon/pikachu` bajo un código `404 Not Found`. El estándar estricto de HTTP dicta el uso del código de estado `405 Method Not Allowed`. Esto indica una falta de manejadores de middleware explícitos para verbos no soportados.

#### API-10 — Inyección de Parámetros de Query Inválidos
*   **Endpoint:** `GET {{base_url}}pokemon?limit=-5`
*   **Métricas:** 200 OK | ~1694ms
*   **Análisis:** El backend adopta un enfoque tolerante/permisivo. En lugar de procesar una validación de tipo o rango de datos en la Query que dispare un error descriptivo (Ej: `400 Bad Request: limit must be positive`), el servicio ignora en silencio el parámetro corrupto, procesa la consulta por defecto y devuelve un `200 OK`. El alto tiempo de ejecución insinúa una degradación del procesamiento por falta de sanitización optimizada en base de datos.

---

## 🛠️ Conclusiones del Reporte Técnico
1.  **Madurez en Caminos Felices:** La API exhibe una estructura e integridad sobresaliente en el acceso transaccional y relacional bajo flujos normales de operación.
2.  **Arquitectura del Ruteo:** El manejo interno de errores expone las capas base del servidor (Express.js), priorizando fallas de emparejamiento de URLs (`400 Bad Request`) y omitiendo respuestas nativas de métodos (`405 Method Not Allowed`).
3.  **Tolerancia Silenciosa:** La omisión de validaciones estrictas sobre tipos o valores ilógicos en parámetros URL puede enmascarar bugs lógicos en desarrollos o integraciones que consuman este servicio de manera productiva.

---

## 🚀 Cómo Ejecutar la Colección en Postman
1. Descarga o clona el archivo `pokeapi_collection.json` provisto en este repositorio.
2. Abre Postman y haz clic en **Import**.
3. Selecciona el archivo JSON descargado.
4. Asegúrate de tener configurada la variable de colección `base_url` con el valor `https://pokeapi.co/api/v2/`.
5. Abre el **Collection Runner** y haz clic en **Run PokeAPI Testing Suite** para validar los scripts automatizados.
