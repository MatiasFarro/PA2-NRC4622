# sistema-pedidos-springboot — PA2

## Tecnologías
Java 21, Spring Boot, Spring MVC, Spring Data JPA, H2, `RestClient` y JUnit 5/Mockito.

## Endpoint de anulaciones
Base URL local: `http://localhost:8081/sistema-pedidos`

- `POST /api/v1/anulaciones` registra una anulación.
- `GET /api/v1/anulaciones` lista las anulaciones.

Ejemplo POST:
```json
{"pedidoId":1,"motivo":"Solicitud del cliente"}
```
La anulación devuelve al inventario la cantidad del pedido, registra auditoría en H2 y cambia el estado a `ANULADO`. Una anulación repetida se rechaza con `409`; un pedido inexistente con `404`; un fallo de Inventario impide guardar la anulación local y se comunica como `503`.

## Configuración de integración
En `src/main/resources/application.properties`, la URL base se configura con `inventario.api.base-url`. El cliente usa timeouts configurados por propiedades. No se hacen reintentos automáticos de POST.

## Ejecución y pruebas
1. Iniciar SQL Server.
2. Desde `Sesión 4/inventario-service`, ejecutar `mvn clean test` y `mvn clean wildfly:run`.
3. Desde esta carpeta ejecutar `./mvnw clean test` (Windows: `mvnw.cmd clean test`) y luego `./mvnw spring-boot:run`.
4. Crear un pedido desde la pantalla `/pedidos` y usar su ID en `requests/pa2-anulaciones.http`.
5. Guardar evidencias de las cinco pruebas funcionales y de ambos comandos `mvn test`.

## Pruebas unitarias PA2
`AnulacionPedidoServiceImplTest` verifica anulación correcta, duplicada y fallo remoto con mocks. Son pruebas unitarias de la lógica de servicio; no sustituyen las pruebas de integración contra WildFly y H2.

## Frontera transaccional
`@Transactional` gestiona las escrituras locales en H2, no la transacción remota de Inventario. Si el servicio remoto devuelve error, no se registra la anulación. Una respuesta remota exitosa seguida de un fallo local puede requerir conciliación; no existe una transacción distribuida atómica.
