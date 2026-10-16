# PA2 — Desarrollo de Aplicaciones Empresariales Avanzado

**Estudiante:** [APELLIDOS Y NOMBRES]  
**Docente:** [NOMBRE DEL DOCENTE]  
**NRC:** [NRC]  
**Periodo:** 2026-20  
**Título:** Integración de devolución de stock y anulación trazable de pedidos

## 1. Objetivo

Ampliar la integración entre `inventario-service` (Jakarta EE) y `sistema-pedidos-springboot` (Spring Boot) mediante un contrato REST de devolución de stock, un cliente HTTP desacoplado y las capas Controller, Service y Repository para registrar anulaciones.

## 2. Modificaciones por proyecto y capa

### Inventario (Jakarta EE)

Se agregó `DevolucionStockRequest` y `DevolucionStockResponse`. `ProductoResource` expone `POST /api/inventario/productos/codigo/{codigo}/devoluciones`. La operación se delega a `InventarioService.devolverStockPorCodigo`, que valida que la cantidad sea positiva, localiza el producto, invoca `ProductoInventario.devolverStock` y fuerza el `flush` dentro de la transacción EJB. Se conserva el campo `@Version` para el control optimista. Las cantidades inválidas deben rechazarse con HTTP 400 y los códigos inexistentes con HTTP 404.

### Pedidos (Spring Boot)

Se añadió `DevolucionStockClient` y `DevolucionStockRestClient`, que usan `RestClient`, DTO de transporte, URL base configurable y manejo controlado de errores HTTP y conexión. Se añadió el enum `PedidoEstado` (`REGISTRADO`, `ANULADO`) al pedido, la entidad `AnulacionPedido` con restricción única por `pedidoId`, `AnulacionPedidoRepository`, `AnulacionPedidoServiceImpl` y `AnulacionPedidoController`.

El endpoint `POST /api/v1/anulaciones` recibe `{"pedidoId":1,"motivo":"Solicitud del cliente"}` y devuelve el registro de auditoría. `GET /api/v1/anulaciones` lista las anulaciones. Una solicitud repetida se rechaza; un pedido inexistente no provoca llamada remota. El cliente remoto se inyecta por constructor, sin instanciarlo con `new`.

## 3. Arquitectura y dependencias

Flujo: `AnulacionPedidoController → AnulacionPedidoServiceImpl → DevolucionStockClient → HTTP/JSON → ProductoResource → InventarioService → ProductoInventario → SQL Server`. En paralelo, `PedidoRepository` y `AnulacionPedidoRepository` persisten pedidos, estados y auditoría en H2. El diagrama editable se encuentra en `docs/diagrama-arquitectura-pa2.md`.

En Jakarta EE se usa `@EJB` para obtener el servicio EJB administrado por WildFly. En Spring se usa Constructor Injection: las dependencias se declaran en el constructor y las resuelve el contenedor IoC. Esto favorece pruebas unitarias con mocks y reduce acoplamiento.

## 4. Pruebas

Se añadieron dos pruebas de dominio para `ProductoInventario.devolverStock`: devolución válida (8 + 2 = 10) y rechazo de cero/negativo sin cambio del stock. Se añadieron tres pruebas unitarias de `AnulacionPedidoServiceImpl`: anulación correcta, rechazo de duplicado sin nueva devolución y fallo remoto sin guardar la auditoría ni cambiar el estado local.

Además, deben ejecutarse y documentarse los cinco escenarios funcionales indicados en `requests/pa2-devoluciones.http` y `requests/pa2-anulaciones.http`: anulación correcta, duplicada, pedido inexistente, inventario no disponible y validación del contrato de devolución. **Completar esta sección con resultados reales de `mvn test`, códigos HTTP, capturas y valores de stock antes/después; no registrar resultados que no se hayan ejecutado.**

## 5. Frontera transaccional y limitaciones

La transacción EJB protege la modificación de stock en SQL Server; `@Transactional` protege las escrituras locales en H2. Son transacciones separadas y no hay atomicidad distribuida. Si Inventario falla antes de responder exitosamente, el servicio no guarda la anulación local. Sin embargo, si Inventario confirma la devolución y después falla la escritura local, el stock remoto ya no puede revertirse automáticamente por la transacción Spring; este caso requiere conciliación o una estrategia compensatoria. No se implementan reintentos automáticos de POST, ya que sin idempotencia podrían duplicar el efecto.

## 6. Referencias (APA)

Eclipse Foundation. (s. f.). *Jakarta RESTful Web Services*. https://jakarta.ee/specifications/restful-ws/

Eclipse Foundation. (s. f.). *Jakarta Enterprise Beans*. https://jakarta.ee/specifications/enterprise-beans/

VMware. (s. f.). *Spring Framework Reference Documentation: REST Clients*. https://docs.spring.io/spring-framework/reference/integration/rest-clients.html

VMware. (s. f.). *Spring Data JPA*. https://docs.spring.io/spring-data/jpa/reference/

## Lista de verificación antes de entregar

- [ ] Completar carátula con NRC, docente y nombre.
- [ ] Ejecutar `mvn test` en ambos proyectos y guardar las salidas reales.
- [ ] Ejecutar los cinco escenarios manuales y añadir capturas propias.
- [ ] Confirmar stock antes/después y estado `ANULADO` en H2.
- [ ] Actualizar el informe con resultados observados.
- [ ] Subir ambos proyectos a un repositorio personal y agregar la URL en la entrega.
