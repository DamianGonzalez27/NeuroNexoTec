# Estructura de respuestas

## 🔏 Regla inamovible
Todas las respuestas de error deben tener el siguiente formato JSON

## 🔋 Interpretacion
Un formato estandarizado para las respuestas de error no es un detalle menor. Es una pieza clave para la observabilidad, trazabilidad, experiencia de desarrollador (DX) y robustez de los sistemas distribuidos. Un buen formato no solo ayuda a automatizar el manejo de errores, sino también a que humanos (equipo de soporte, QA, DevOps, frontend) puedan entender y actuar sobre el error eficientemente.

## ⚙️ Estructuras

#### Estructura ```Body Response``` simple 
```json
{
  "timestamp": "2025-05-29T20:00:00Z",
  "status": 500,
  "error": "Internal Server Error",
  "code": "USR_001",
  "message": "Error con la comunicacion del servidor",
  "path": "/api/v1/users",
  "traceId": "123e4567-e89b-12d3-a456-426614174000"
}
```

#### Estructura ```Body Response``` validaciones
```json
{
  "timestamp": "2025-05-29T20:00:00Z",
  "status": 422,
  "error": "Validation error",
  "code": "USR_001",
  "message": "Error de validacion de campos",
  "path": "/api/v1/users",
  "traceId": "123e4567-e89b-12d3-a456-426614174000",
  "errors": {
    "fieldName": {
        "message": "[fieldName] no puede ser vacio."
    }
  }
}
```

## 🔍 Explicación de cada campo


| Campo	| Tipo	| Descripción | 
|:-------:|:-------:|-------------|
| **timestamp** | ```string``` | (ISO 8601) |	Marca exacta del momento en que ocurrió el error. Útil para correlacionar con logs, trazas distribuidas y debugging temporal.|
| **status** | ```number``` | Código HTTP numérico que describe el tipo de error (por ejemplo: 400, 404, 500). Es lo que evalúan los clientes y gateways.|
| **error** | ```string``` | Descripción textual corta del error HTTP (por ejemplo: "Bad Request", "Unauthorized"). Útil para debug rápido o lectura por humanos.|
| **code** | string | Código interno del sistema, único y estandarizado (por ejemplo: USR_001, AUTH_403_001). Es el identificador principal del tipo de error para el backend, frontend, documentación, métricas, y | analítica.|
| **message** | ```string``` |Mensaje legible explicando la causa del error, pensado para ser mostrado al usuario final o a un desarrollador. Debe ser claro y conciso.|
| **path** | ```string``` | El endpoint que generó el error. Facilita el rastreo sin necesidad de logs adicionales. Ideal para sistemas con múltiples APIs.|
| **requestId** | ```string```| UUID generado en cada request entrante (por gateway o interceptor). Permite correlación 1:1 entre logs, errores, trazas y métricas. Fundamental en entornos concurrentes y distribuidos.|
| **errors** | ```object```| Es un objeto que brindara informacion puntual sobre errores de validacion de campos en un request del tipo POST por ejemplo|


## 🧠 Justificación del Formato

### Trazabilidad y Observabilidad
- ``timestamp``, ``path`` y ``requestId`` permiten rastrear un error desde una alerta en Grafana hasta el log detallado en Elasticsearch o CloudWatch, pasando por el trace en OpenTelemetry.

### Manejo programático de errores
- ``code`` es clave para que el frontend o cualquier cliente automatice respuestas. Por ejemplo: si code = AUTH_401_001, mostrar login; si code = USR_001, mostrar error de formulario.

### Estandarización trans-API y multi-servicio
- En sistemas con muchos microservicios, compartir un mismo esquema evita inconsistencias, bugs y horas de debugging porque ya se conoce el contrato.

### Mejor experiencia para QA, soporte y frontend
- Saber qué ocurrió y por qué, sin necesidad de abrir los logs o tener acceso a infraestructura. El código y mensaje permiten crear una wiki de errores mantenible.





## 4. Propagación de Trace ID

* El sistema debe aceptar el header `X-Trace-Id`.
* Si no viene, debe generarse un UUID v4 por petición.
* Debe propagarse por todos los logs, respuestas de error y llamadas internas.

## 5. Logging Estructurado

* Toda excepción capturada debe ser logueada.
* El log debe estar en formato JSON con:

  * `traceId`
  * `type`: tipo de error
  * `exception`: clase de excepción
  * `message`: detalle
  * `stacktrace`: solo si está en entorno de desarrollo

## 6. Compatibilidad Multilenguaje

Se deben implementar SDKs que respeten este contrato en:

* Java (Spring Boot)
* Node.js (Express, Fastify)
* Python (FastAPI, Flask)
* Go (Gin, Echo)

## 7. Documentación del Contrato de Errores

* Se debe incluir en la especificación OpenAPI (Swagger).
* Todos los endpoints deben declarar sus posibles respuestas de error según este estándar.

## 8. Validación Automática

* Se recomienda usar linters, pruebas o validadores que aseguren el cumplimiento del estándar en CI/CD.

## 9. Observabilidad

Este estándar está diseñado para integrarse de forma natural con:

* OpenTelemetry
* Grafana Loki
* Sentry
* Datadog
* Zipkin / Jaeger

## 10. Comunidad y Futuro

Este documento está en fase de borrador. Está abierto a iteraciones junto a la comunidad.

> Si deseas participar, sugerir mejoras o implementar este estándar en tu organización, contáctanos. Estamos construyendo una suite de herramientas, SDKs y guías para que la adopción sea inmediata y eficaz.

---

**Versión**: 0.1
**Autor**: Damian + Lumen
**Licencia**: NeuroNexo Tec - Uso libre, contribución controlada
