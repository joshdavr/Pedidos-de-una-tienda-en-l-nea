# ADR-001 — Saga orquestada dentro del monolito, con reserva primero y compensaciones

| | |
|---|---|
| **Estado** | Aceptada |
| **Tipo** | arquitectónica |
| **Fecha** | 2026-10-04 |
| **Ejercicio** | Ejercicio 4 — Pedidos de una tienda en línea |

## Contexto

Pedido, inventario y pago deben quedar coherentes aunque el pago llegue tarde, duplicado o no llegue; no hay transacción distribuida posible con la pasarela.

## Decisión

El módulo Pedidos orquesta una **saga** con máquina de estados explícita: reservar → cobrar → confirmar reserva → preparar → despachar. Cada paso con efecto externo tiene su compensación (liberar reserva, reembolsar). Todo corre en un monolito modular; los efectos externos salen por outbox + RabbitMQ.

## Alternativas consideradas

- Coreografía entre microservicios (rechazada: más difícil de razonar y depurar a esta escala)
- Reservar solo después del pago (rechazada: cobros sin stock)
- Transacción única (imposible con la pasarela).

## Consecuencias

- (+) flujo legible y auditable, fallos tratados de forma explícita, módulos separables más adelante.
- (−) hay que diseñar y probar cada compensación; el monolito debe mantener sus fronteras internas.
