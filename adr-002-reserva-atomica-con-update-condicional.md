# ADR-002 — Reserva de inventario con `UPDATE` condicional atómico en PostgreSQL

| | |
|---|---|
| **Estado** | Aceptada |
| **Tipo** | tecnológica |
| **Fecha** | 2026-10-04 |
| **Ejercicio** | Ejercicio 4 — Pedidos de una tienda en línea |

## Contexto

Varios clientes pueden comprar el mismo producto a la vez desde la web y las redes sociales; no puede haber sobreventa.

## Decisión

La reserva se hace con una sola sentencia: `UPDATE stock SET reservado = reservado + :q WHERE sku = :s AND (disponible - reservado) >= :q`; si afecta 0 filas, no hay stock. Cada reserva se guarda con pedido, cantidad y vencimiento; un worker libera las vencidas. Confirmar y liberar usan la clave `(pedido_id, sku)` para ser idempotentes.

## Alternativas consideradas

- Contador en Redis (rechazada: segunda fuente de verdad que puede divergir)
- Bloqueo optimista por versión (viable, pero con reintentos frecuentes en picos)
- Bloqueo pesimista sobre la fila (rechazada: contención).

## Consecuencias

- (+) correcto bajo concurrencia, simple y sin infraestructura nueva.
- (−) la fila de stock de productos muy populares es un punto caliente; si el volumen crece se separa Inventario y se particiona o se usa un contador por lotes.
