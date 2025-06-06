# Error Handling Standard - v0.1
## 🧭 Introducción

En los sistemas distribuidos modernos —y especialmente en microservicios como nuestro ```Notification Service```— los errores no son una posibilidad: son una certeza. Ya sea por un fallo en una base de datos, un proveedor de correo caído o un bug inesperado en una librería de terceros, los errores forman parte del ciclo de vida de cualquier sistema en producción.

Esta guía te mostrará cómo diseñar e implementar un sistema de manejo de errores centralizado, robusto y reutilizable, que transforme fallos silenciosos o caóticos en respuestas claras, rastreables y comprensibles tanto para el equipo de desarrollo como para los usuarios finales o integradores del sistema.

❗ ¿Por qué es importante aplicar un Error Handler?
Un error handler bien diseñado no solo captura excepciones: estructura y comunica los errores, proporciona trazabilidad, y garantiza una experiencia de desarrollo más profesional y mantenible.

## Escenarios comunes donde es clave:

📬 Servicios de terceros fallan: como SMTP, Twilio o AWS SES. Necesitás capturar el error y responder de forma coherente al cliente.

🧾 Errores de validación: un input malformado puede terminar en una NullPointerException si no se maneja adecuadamente.

⚠️ Fallos de lógica interna: como intentos de reenvío a direcciones inválidas o proveedores no soportados.

🔐 Errores de autenticación/autorización: por falta de credenciales válidas o permisos insuficientes.

Sin un handler claro, tus APIs terminan lanzando trazas de pila crudas (stacktraces), mensajes confusos o errores silenciosos que son un infierno para debuggear y una pesadilla para tus usuarios.

## 🌟 Beneficios de tener un Error Handler centralizado
1. Uniformidad de respuestas
Todas las APIs responden con una estructura estándar, por ejemplo:

~~~json
{
  "timestamp": "2025-05-29T15:30:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "El campo 'to' no puede estar vacío.",
  "traceId": "9e582edc-087a-4a9e-b8d6-b7a88fc5a935"
}
~~~

2. Trazabilidad mejorada
Cada error incluye un traceId único (usualmente pasado como header) para correlacionar eventos en logs, métricas y dashboards.

3. Facilidad para el monitoreo y alertas
Con este sistema podés integrar herramientas como Grafana, Prometheus o Sentry, y disparar alertas ante ciertos tipos de errores.

4. Menor acoplamiento con clientes
Los clientes de tus APIs no necesitan interpretar errores según el tipo de excepción Java: les das un contrato claro de qué esperar.

## 🛠️ Pasos para implementar el Error Handler
✅ 1. Definí tus errores personalizados
~~~java
public class EmailSendException extends RuntimeException {
    public EmailSendException(String message) {
        super(message);
    }
}
~~~
Opcionalmente podés crear una jerarquía como ApplicationException con códigos y niveles de severidad.

✅ 2. Implementá un handler global
Usá @ControllerAdvice y @ExceptionHandler:

~~~java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmailSendException.class)
    public ResponseEntity<ErrorResponse> handleEmailSend(EmailSendException ex) {
        return buildResponse(HttpStatus.SERVICE_UNAVAILABLE, ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        return buildResponse(HttpStatus.INTERNAL_SERVER_ERROR, "Error inesperado.");
    }

    private ResponseEntity<ErrorResponse> buildResponse(HttpStatus status, String message) {
        return new ResponseEntity<>(
            new ErrorResponse(
                Instant.now().toString(),
                status.value(),
                status.getReasonPhrase(),
                message,
                RequestContext.getRequestId()
            ),
            status
        );
    }
}
~~~

✅ 3. Diseñá un modelo estándar para las respuestas de error

~~~java
@Data
@AllArgsConstructor
public class ErrorResponse {
    private String timestamp;
    private int status;
    private String error;
    private String message;
    private String traceId;
}
~~~

✅ 4. Usá RequestContext para mantener el traceId
Este componente se puede poblar en un filtro que lea el header:

~~~java
@Component
public class TraceIdFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {
        String traceId = request.getHeader("X-Trace-Id");
        RequestContext.setRequestId(traceId != null ? traceId : UUID.randomUUID().toString());
        try {
            filterChain.doFilter(request, response);
        } finally {
            RequestContext.clear();
        }
    }
}
~~~

✅ 5. Integra los logs con el traceId
Tu clase LogObserver o Logger custom debería incluir el traceId en cada mensaje de log para facilitar el seguimiento.

🤝 Unite a la comunidad o contactanos
Si llegaste hasta acá, probablemente ya estés diseñando o manteniendo una API seria. En ese camino, no estás solo.

Si querés ayuda para implementar tu sistema de errores, conectar con otros desarrolladores o simplemente resolver un bloqueo técnico, podés contactarnos o sumarte a nuestra comunidad.

📬 ¿Dudas puntuales? Escribinos.
🧑‍💻 ¿Querés compartir lo que hiciste? ¡Mostralo en la comunidad!
🔧 ¿Querés una consultoría a medida? Podemos trabajar juntos en eso también.