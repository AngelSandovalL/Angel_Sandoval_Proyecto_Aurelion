# Summary

Este documento proporciona una descripción técnica exhaustiva de un dataset transaccional diseñado para la analítica de negocio. El dataset modela las entidades fundamentales de un dominio retail: Clientes, Productos y las transacciones de venta que los interconectan. El objetivo es la explotación de estos datos para la generación de insights accionables y la optimización de estrategias comerciales.

# Arquitectura del Modelo de Datos

El dataset esta implementado bajo un **esquema de estrella**. El modelo se compone de:
* Una tabla de hechos central: `detalle_ventas`, que registra las métricas cuantitativas del negocio (`cantidad`, `importe`) con la maxima granularidad (nivel de linea de producto por transacción).
* Múltiples tablas de dimensiones: `clientes`, `productos` y `ventas` que proveen el contexto descriptivo para las métricas.

Esta estructura esta optimizada para la agregación y el desgloce (slice and dice) de métricas a través de sus dimensiones.

El dataset esta conformado por 4 archivos binarios .xlsx que se omiten mediante el archivo .gitignore para mantener un tamaño pequeño en el repositorio. Dichos binarios pueden ser encontrados en el siguiente enlace seguro: https://drive.google.com/drive/folders/12qR9v6YugHXqirusb8ufgwqtYiAb8lLg?usp=sharing 

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

## Dimensión: `clientes`

| Atributo | Descripción | Tipo de Dato (SQL) | Escala |
|----------|-------------|--------------------|--------|
|`id_cliente`|PK. Identificador unico del cliente.|`INT`|Nominal|
|`nombre_cliente`|Nombre completo del cliente|`VARCHAR`|Nominal|
|`email`|Dirección de correo electrónico|`VARCHAR`|Nominal|
|`ciudad`|Atributo geográfico del cliente|`VARCHAR`|Nominal|
|`fecha_alta`|Timestamp de registro del cliente|`DATE`|Intervalo|

## Dimensión: `productos`

| Atributo | Descripción | Tipo de Dato (SQL) | Escala |
|----------|-------------|--------------------|--------|
|`id_producto`|PK. Identificador único del producto|`INT`|Nominal|
|`nombre_producto`|Descripcion del producto|`VARCHAR`|Nominal|
|`categoria`|Jerarquía de clasificación de producto|`VARCHAR`|Nominal|
|`precio_unitario`|Precio de lista del producto|`DECIMAL`|Razón|

## Dimensión: `ventas`

| Atributo | Descripción | Tipo de Dato (SQL) | Escala |
|----------|-------------|--------------------|--------|
|`id_ventas`|PK. Identificador único de transacción|`INT`|Nominal|
|`fecha`|Timestamp de la transacción|`DATE`|Intervalo|
|`id_cliente`|FK. Referencia a la dimensión `clientes`|`INT`|Nominal|
|`nombre_cliente`|Atributo redundante|`VARCHAR`|Nominal|
|`email`|Atributo redundante|`VARCHAR`|Nominal|
|`medio_pago`|Método de pago utilizado|`VARCHAR`|Nominal|

## Hechos: `detalle_ventas`

| Atributo | Descripción | Tipo de Dato (SQL) | Escala |
|----------|-------------|--------------------|--------|
|`id_venta`|FK. Referencia a la tabla `ventas`|`INT`|Nominal|
|`id_producto`|FK. Referencia a la tabla `productos`|INT|Nominal|
|`nombre_producto`|Atributo redundante|`VARCHAR`|Nominal|
|`cantidad`|Métrica: Unidades vendidas|`INT`|Razón|
|`precio_unitario`|Precio al momento de la transacción|`DECIMAL`|Razón|
|`importe`|Métrica calculada (`cantidad` * `precio`)|`DECIMAL`|Razón|

# Consideraciones técnicas

* **Granularidad:** La unidad más pequeña reside en la tabla `detalle_ventas`, donde cada registro representa un SKU único dentro de una transacción específica.
* **Alcance Temporal:** El scope temporal de los datos de ventas, abarca desde 2024-06-28 hasta 2024-01-02, sin embargo, es posible encontrar clientes registrados en fechas previas.

# Calidad de los Datos y Oportunidades Analíticas

Durante la exploración del dataset se ha detectado redundancia en las tablas `ventas` y `detalle_ventas`, indicando un grado de desnormalización. Atributos como `nombre_cliente` o `precio_unitario` estan replicados comprometiendo la integridad de los datos y la eficiencia del almacenamiento, sin embargo, es posible que se decida mantenerlos como redundancia controlada para facilitar algunas operaciones, la conclusión de esta decisión se definira durante el tratamiento de los datos y quedara documentada en el código.

## Casos de Uso Analíticos Potenciales

Se ha identificado que el datset es ideal para una variedad de análisis:

* **Descriptivo y KPIS:**
    * Análisis de rendimiento de producto y categorías.
    * Segmentacion de clientes por valor (Análisis RFM: Recencia, Frecuencia, Monetario).
    * Análisis de patrones de pago y tendencias geográficas.
    * Análisis de series temporales de ventas.

* **Modelado Predictivo y Avanzado:**
    * Pronóstico de Demanda: Modelos de series de tiempo (ARIMA, Prophet) para predecir ventas futuras.
    * Sistema de Recomendación: Modelos de filtrado colaborativo o basados en contenido para sugerir productos.
    * Análisis de Cesta de Compra: Aplicación de algoritmos (ej. Apriori) para encontrar reglas de asociación entre productos.
    * Modelado de Churn Rate: Modelos de clasificación para predecir la probabilidad de fuga de clientes.
    * Customer Lifetime Value (CLV): Modelos predictivos para estimar el valor total que un cliente aportará a la empresa.