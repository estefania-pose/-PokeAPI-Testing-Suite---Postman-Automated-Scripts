## Analisis Detallado de Ejecucion

### Escenarios Positivos

<!-- API-01 -->


<details>
<summary><b>API-01 - Obtener un Pokemon existente por Nombre</b></summary>
<br>

* **Endpoint:** `GET {{base_url}}pokemon/pikachu`
* **Metricas:** 200 OK | ~458ms | 3/3 tests Pass
* **Que paso:** la API devolvio el recurso completo de Pikachu, incluyendo el campo `abilities` esperado.
* **Analisis:** comportamiento correcto y esperable para un recurso valido consultado por nombre.

**Evidencia en Postman:**

<img width="709" height="422" alt="poke1" src="https://github.com/user-attachments/assets/af52bba6-6e39-44c0-afad-d969f283a9cc" />


</details>
<br>

<!-- API-02 -->
<details>
<summary><b>API-02 - Obtener un Pokemon por ID Numerico</b></summary>
<br>

* **Endpoint:** `GET {{base_url}}pokemon/6`
* **Metricas:** 200 OK | ~323ms | 2/2 tests Pass
* **Que paso:** la API acepto el ID numerico (6 = Charizard) de la misma forma que acepto el nombre en API-01.
* **Analisis:** confirma que el endpoint soporta ambos identificadores de forma transparente, un buen indicio de diseño flexible de la API.

**Evidencia en Postman:**

<img width="709" height="400" alt="poke2" src="https://github.com/user-attachments/assets/ca3b1a19-4a4b-4752-a93d-c7bce55dca47" />


</details>
<br>

<!-- API-03 -->
<details>
<summary><b>API-03 - Listado Paginado de Recursos</b></summary>
<br>

* **Endpoint:** `GET {{base_url}}pokemon?limit=20&offset=0`
* **Metricas:** 200 OK | ~114ms | 3/3 tests Pass
* **Que paso:** devolvio exactamente 20 resultados y un campo `next` con la URL de la siguiente pagina.
* **Analisis:** la paginacion funciona como se espera de un endpoint de listado bien diseñado (count, next, previous, results).

**Evidencia en Postman:**

<img width="709" height="435" alt="poke3" src="https://github.com/user-attachments/assets/8015a349-b78a-4742-b8d2-e556341889cf" />


</details>
<br>

<!-- API-04 -->
<details>
<summary><b>API-04 - Consultar Relaciones de Recursos (Tipo)</b></summary>
<br>

* **Endpoint:** `GET {{base_url}}type/electric`
* **Metricas:** 200 OK | ~164ms | 2/2 tests Pass
* **Que paso:** devolvio la lista de Pokemon asociados al tipo "electric".
* **Analisis:** valida correctamente la relacion entre recursos (tipo → coleccion de pokemon), tipico de una API bien normalizada.

**Evidencia en Postman:**

<img width="709" height="355" alt="poke4" src="https://github.com/user-attachments/assets/043738b4-99c5-48fb-a874-21b62020c746" />


</details>
<br>

<!-- API-05 -->
<details>
<summary><b>API-05 - Acuerdo de Nivel de Servicio (SLA) y Headers</b></summary>
<br>

* **Endpoint:** `GET {{base_url}}ability/overgrow`
* **Metricas:** 200 OK | ~124ms | 2/2 tests Pass
* **Que paso:** respondio muy por debajo del umbral de 1000ms definido, con `Content-Type: application/json`.
* **Analisis:** buen indicio de performance del servicio para consultas simples de un solo recurso.

**Evidencia en Postman:**

<img width="709" height="398" alt="poke5" src="https://github.com/user-attachments/assets/6b1e006f-dc22-499e-a7d7-b9a26b5c0a42" />


</details>

---

### Escenarios Negativos y Hallazgos Clave de QA

<!-- API-06 -->
<details>
<summary><b>API-06 - Recurso Inexistente (ID Fuera de Rango)</b></summary>
<br>

* **Endpoint:** `GET {{base_url}}pokemon/999999`
* **Metricas:** 404 Not Found | ~1647ms | 1/1 test Pass
* **Que paso:** la API reconocio que la ruta era valida (`/pokemon/:id`) pero que el recurso puntual no existe, devolviendo 404 tal como se esperaba.
* **Analisis:** comportamiento correcto segun el estandar REST — 404 se reserva para "ruta valida, recurso no encontrado".
* **Nota adicional:** el tiempo de respuesta (1647ms) fue notablemente mas alto que en las consultas de recursos existentes, posiblemente porque una busqueda fallida no se beneficia de la misma estrategia de cache (`cf-cache-status: MISS`) que si tienen los recursos validos.

**Evidencia en Postman:**

<img width="662" height="319" alt="poke6" src="https://github.com/user-attachments/assets/62edd539-3554-407b-af4e-784ecb84f7cf" />


</details>
<br>

<!-- API-07 -->
<details>
<summary><b>API-07 - Nombre de Pokemon Mal Escrito</b></summary>
<br>

* **Endpoint:** `GET {{base_url}}pikachuu`
* **Metricas:** 400 Bad Request | ~261ms | 1/2 tests (el test de status Fallo)
* **Que paso:** se esperaba 404, pero la API devolvio 400. El segundo test (que el cuerpo no contenga datos de pokemon) si paso.
* **Analisis — hallazgo clave:** PokeAPI distingue entre 400 ("la ruta no matchea ningun patron valido registrado") y 404 ("la ruta es valida pero el recurso especifico no existe"). Como `pikachuu` no encaja en ningun patron de ruta reconocido por el router, cae en 400 en vez de 404. Es una decision de diseño valida, aunque no es la mas comun en APIs REST (muchas devuelven 404 para cualquier "no encontrado", sin distinguir el nivel de ruta vs. recurso).

**Evidencia en Postman:**

<img width="709" height="364" alt="poke7" src="https://github.com/user-attachments/assets/530b87bc-0bbf-4608-88f6-f415b2843085" />


</details>
<br>

<!-- API-08 -->
<details>
<summary><b>API-08 - Endpoint Inexistente (Path Mal Escrito)</b></summary>
<br>

* **Endpoint:** `GET {{base_url}}pokemonn/1`
* **Metricas:** 400 Bad Request | ~452ms | 0/1 test (Fallo)
* **Que paso:** mismo patron que API-07 — se esperaba 404, la API devolvio 400.
* **Analisis:** confirma que el comportamiento de API-07 no fue un caso aislado, sino una regla consistente del servicio: cualquier ruta que no matchea ningun patron definido (ni siquiera parcialmente) devuelve 400, no 404.

**Evidencia en Postman:**

<img width="709" height="384" alt="poke8" src="https://github.com/user-attachments/assets/f1c3a255-2e5c-45dd-9f3f-f646d770f2c0" />


</details>
<br>

<!-- API-09 -->
<details>
<summary><b>API-09 - Metodo HTTP No Soportado</b></summary>
<br>

* **Endpoint:** `POST {{base_url}}pokemon/pikachu`
* **Metricas:** 404 Not Found | ~219ms | 0/1 test (Fallo, se esperaba 405)
* **Que paso:** al mandar POST a una ruta que solo tiene GET registrado, la API respondio con 404 y el body `Cannot POST /api/v2/pokemon/pikachu` (mensaje caracteristico de Express.js).
* **Analisis — hallazgo clave:** lo tecnicamente correcto segun el estandar HTTP seria devolver 405 Method Not Allowed (la ruta existe, pero no admite ese metodo), no 404 (que implica que el recurso no existe en absoluto). Este es un patron comun en APIs construidas sobre Express sin un manejador explicito de metodos no soportados por ruta — Express, por defecto, no encuentra ninguna ruta que combine metodo + path, y responde como si la ruta no existiera. Es una observacion valida de calidad de API, util para comparar con otras herramientas (por ejemplo, esta misma prueba en Fravega/DummyJSON podria comportarse distinto).

**Evidencia en Postman:**

<img width="709" height="336" alt="poke9" src="https://github.com/user-attachments/assets/01131ec8-20d9-48c4-b0a8-8f9254912714" />


</details>
<br>

<!-- API-10 -->
<details>
<summary><b>API-10 - Inyeccion de Parametros de Query Invalidos</b></summary>
<br>

* **Endpoint:** `GET {{base_url}}pokemon?limit=-5`
* **Metricas:** 200 OK | ~1694ms | 1/1 test Pass
* **Que paso:** en vez de rechazar el valor negativo, la API lo ignoro silenciosamente y devolvio igualmente una respuesta 200 con datos (aparentemente sin aplicar el limite negativo, trayendo un listado igual).
* **Analisis:** la API es "permisiva" ante parametros invalidos — no rompe (no da 500), pero tampoco valida ni informa al cliente que el parametro fue ignorado. Desde una perspectiva de diseño de API, seria mas informativo devolver un 400 explicando que `limit` debe ser positivo, en vez de aplicar un comportamiento silencioso no documentado. El tiempo de respuesta tambien fue el mas alto de toda la tanda (1694ms), lo que sugiere que proceso la consulta igual, sin optimizar por el parametro invalido.

**Evidencia en Postman:**

<img width="709" height="344" alt="poke10" src="https://github.com/user-attachments/assets/d3729a50-f2ec-46db-8397-37c1bad61a56" />


</details>
