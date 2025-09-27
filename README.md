# Angel_Sandoval_Proyecto_Aurelion

# Summary

Este documento proporciona una descripción técnica exhaustiva de un dataset transaccional diseñado para la analítica de negocio. El dataset modela las entidades fundamentales de un dominio retail: Clientes, Productos y las transacciones de venta que los interconectan. El objetivo es la explotación de estos datos para la generación de insights accionables y la optimización de estrategias comerciales.

# Arquitectura del Modelo de Datos

El dataset esta implementado bajo un **esquema de estrella**. El modelo se compone de:
* Una tabla de hechos central: `detalle_ventas`, que registra las métricas cuantitativas del negocio (`cantidad`, `importe`) con la maxima granularidad (nivel de linea de producto por transacción).
* Múltiples tablas de dimensiones: `clientes`, `productos` y `ventas` que proveen el contexto descriptivo para las métricas.

Esta estructura esta optimizada para la agregación y el desgloce (slice and dice) de métricas a través de sus dimensiones.

# Cardinalidad e integridad referencial

Las relaciones entre las tablas se establecen mediante llaves primarias (PK) y llaves foráneas (FK), definiendo la integridad referencial del modelo con la siguiente cardinalidad:

* Relación `clientes` → `ventas` (1:*): Un cliente puede realizar múltiples ventas, pero cada venta pertenece a un único cliente.
    * `clientes` (PK: `id_cliente`) 1:* `ventas` (FK: `id_ventas`)

* Relación `ventas` → `detalle_ventas` (1:*): Una venta se decompone en múltiples lineas de detalle (productos), pero cada linea de detalle pertenece a una sola venta.
    * `ventas` (PK:`id_venta`) 1:* `detalle_ventas` (FK: `id_venta`)

* Relación `productos` → `detalle_ventas` (1:*): Un producto puede aparecer en múltiples lineas de detalle a través de diferentes ventas.
    * `productos` (PK: `id_producto`) 1:* `detalle_ventas` (FK: `id_producto`)

La tabla `detalle ventas` actua como la tabla de hechos que resuelve la relación lógica *:* entre las entidades `ventas` y `productos`.

# Data Dictionary

A continuación, se detalla el esquema de cada tabla, especificando los atributos, tipos de datos y su escala de medición estadística.

## Dimension: `clientes`

| Atributo | Descripción | Tipo de Dato | Escala |
|----------|-------------|--------------|--------|



