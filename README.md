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
