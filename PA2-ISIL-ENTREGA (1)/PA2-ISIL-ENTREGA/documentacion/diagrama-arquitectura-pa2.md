# Diagrama de arquitectura PA2

```mermaid
flowchart LR
  AC[Cliente HTTP / Postman] --> C[AnulacionPedidoController]
  C --> S[AnulacionPedidoServiceImpl]
  S --> PR[PedidoRepository]
  S --> AR[AnulacionPedidoRepository]
  PR --> H2[(H2: pedidos y estado)]
  AR --> H2
  S --> DC[DevolucionStockClient]
  DC --> RC[DevolucionStockRestClient]
  RC -->|HTTP / JSON POST devoluciones| R[ProductoResource]
  R --> IS[InventarioService @Stateless]
  IS --> PI[ProductoInventario JPA]
  PI --> SQL[(SQL Server InventarioDB)]
```

La transacción Spring controla las escrituras locales en H2. Inventario administra su propia transacción en SQL Server. No existe una transacción distribuida atómica entre ambos sistemas.
