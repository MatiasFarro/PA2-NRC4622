# PA2 ISIL — paquete de trabajo

Este ZIP contiene los dos proyectos requeridos por la PA2 y la documentación base.

## Contenido
- `inventario-service/`: Jakarta EE, endpoint de devolución de stock y pruebas de dominio.
- `sistema-pedidos-springboot/`: cliente REST, flujo de anulación, auditoría H2 y pruebas con Mockito.
- `documentacion/`: diagrama, borrador del informe y lista de verificación.

## Orden de arranque
1. Configurar SQL Server según el README de `inventario-service`.
2. Ejecutar las pruebas de Inventario y arrancar WildFly en puerto 8082.
3. Ejecutar las pruebas de Spring Boot y arrancar Pedidos en puerto 8081.
4. Crear un pedido real desde `/sistema-pedidos/pedidos`, anular su ID usando `requests/pa2-anulaciones.http` y guardar capturas de los cinco escenarios.

## Importante antes de entregar
No pude ejecutar Maven aquí porque el wrapper no pudo descargar Maven desde Maven Central en este entorno. Por tanto, no se afirma que los tests hayan pasado. Ejecuta los comandos Maven localmente y añade las capturas reales y la salida de las pruebas. Completa los campos personales del informe, genera el PDF final con tus evidencias y sube ambos proyectos a un repositorio Git personal accesible para el docente.
