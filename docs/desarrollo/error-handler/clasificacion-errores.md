# Clasificación de Errores

Los errores no son simplemente "algo salió mal". En un sistema distribuido y mantenible, cada error debe tener un contexto semántico claro y accionable. Para lograrlo, los errores deben clasificarse en categorías estructuradas, las cuales no solo permiten un mejor debugging, sino también facilitan:

- Automatización del manejo de errores en clientes (frontend, SDKs)
- Alertas e indicadores en observabilidad
- Documentación técnica y soporte
- Diagnóstico rápido para DevOps y SRE

## Errores del cliente - 4xx

 **Qué son**: Solicitudes mal formadas, inválidas o no autorizadas.

 **Responsable**: El consumidor del servicio.

 **Ejemplos comunes**:

- Formatos incorrectos
- Faltan parámetros
- Violaciones de negocio por parte del cliente

| Código HTTP | Código Interno     | Descripción                           | Utilidad                           |
| :-----------: | ------------------ | ------------------------------------- | ---------------------------------- |
| 400         | `VALIDATION_001`   | Cuando un campo no es correcto o no cumple con validaciones del request       | Validacion y renderizado de errores del canal consumidor    |
| 401         | `AUTH_401_001`     | No autenticado                        | Evitar accesos no controlados al sistema                 |
| 403         | `AUTH_403_001`     | No autorizado para acceder al recurso | Evitar accesos espeficicos no autorizados al sistema |
| 404         | `RESOURCE_404_001` | Recurso no encontrado                 | Dar feedback al usuario       |
| 409         | `CONFLICT_409_001` | Conflicto de datos (e.g., ya existe)  | Evita reintentos innecesarios      |
| 422         | `BUSINESS_422_001` | Errores que surgen de la validacion de reglas de negocio | Muestran mensajes logicos referentes a la logica de negocio al canal consumidor |


✅ Importancia:
Permiten que el cliente sepa que debe corregir su solicitud, sin necesidad de escalar el error.

## Errores del servidor - 5xx

 **Qué son**: Fallos internos del sistema.

 **Responsable**: El backend o algún sistema interno.

 **Ejemplos comunes**:

- Null pointers
- Timeouts
- Faltas de conectividad a servicios internos

| Código HTTP | Código Interno       | Descripción                           | Utilidad                                  |
| ----------- | -------------------- | ------------------------------------- | ----------------------------------------- |
| 500         | `INTERNAL_500_001`   | Error no controlado                   | Sirve como catch-all, debe estar logueado |
| 502         | `DOWNSTREAM_502_001` | Servicio externo no disponible        | Retentable por el cliente                 |
| 503         | `TIMEOUT_503_001`    | Timeout con proveedor o base de datos | Retry automático o fallback               |
| 504         | `GATEWAY_504_001`    | Timeout en API Gateway                | Se pueden analizar cuellos de botella     |

✅ Importancia:
Estos errores deben ser siempre logueados con requestId y generan alertas. En sistemas resilientes se diseñan estrategias de retry o fallback sobre ellos.


Los errores deben clasificarse en los siguientes tipos:

* `VALIDATION_ERROR` (400)
* `BUSINESS_RULE_ERROR` (422)
* `AUTHENTICATION_ERROR` (401)
* `AUTHORIZATION_ERROR` (403)
* `RESOURCE_NOT_FOUND` (404)
* `CONFLICT` (409)
* `INTERNAL_ERROR` (500)
* `SERVICE_UNAVAILABLE` (503)