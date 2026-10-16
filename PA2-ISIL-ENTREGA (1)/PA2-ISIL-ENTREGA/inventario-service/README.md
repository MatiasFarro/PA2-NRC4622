# inventario-service — PA2

## Tecnologías
Java 21, Jakarta EE 11, Jakarta REST, EJB (`@Stateless`), JPA, WildFly 41 y SQL Server.

## Endpoint nuevo: devolución de stock
- Método: `POST`
- URL local: `http://localhost:8082/inventario-service/api/inventario/productos/codigo/{codigo}/devoluciones`
- Body: `{"cantidad": 2}`
- Respuesta `200`: `{"codigo":"LAP-001","cantidadDevuelta":2,"stockRestante":10}`

La cantidad debe ser positiva. Una cantidad no válida se responde como `400`; un código no existente como `404`. La operación se ejecuta dentro de una transacción local de Inventario y conserva `@Version` para el bloqueo optimista.

## Ejecución
1. Configura SQL Server y la unidad de persistencia `InventarioPU`.
2. Desde esta carpeta ejecuta `mvn clean test`.
3. Ejecuta `mvn clean wildfly:run`; la configuración del proyecto usa el puerto HTTP 8082.
4. Prueba las solicitudes de `requests/pa2-devoluciones.http` o `requests/inventario-service.http`.

## Pruebas PA2
`ProductoInventarioTest` incluye pruebas para devolución válida y devolución con cantidad cero/negativa, verificando que el stock no cambie ante entradas inválidas.

## Nota
No se debe acceder a `InventarioDB` desde Spring Boot. Las modificaciones de stock deben pasar por esta API REST. Ejecuta `mvn test` en esta carpeta para generar resultados reales.
