# Documentación SQL – Sistema de Ventas (Taller Parte 2)

## Tablas

### `clientesO`
Registra los clientes del sistema.

| Columna         | Tipo         | Descripción                  |
|-----------------|--------------|------------------------------|
| id              | SERIAL       | Clave primaria               |
| nombre          | VARCHAR(100) | Nombre del cliente           |
| correo          | VARCHAR(100) | Correo electrónico           |
| fecha_registro  | DATE         | Fecha de registro            |

---

### `productosO`
Catálogo de productos disponibles para la venta.

| Columna         | Tipo           | Descripción                  |
|-----------------|----------------|------------------------------|
| id              | SERIAL         | Clave primaria               |
| nombre          | VARCHAR(100)   | Nombre del producto          |
| precio          | NUMERIC(10, 2) | Precio unitario              |
| stock           | INT            | Stock disponible             |

---

### `ventasO`
Registra cada venta realizada.

| Columna         | Tipo    | Descripción                                  |
|-----------------|---------|----------------------------------------------|
| id              | SERIAL  | Clave primaria                               |
| cliente_id      | INT     | Referencia a `clientesO(id)`                 |
| producto_id     | INT     | Referencia a `productosO(id)`                |
| cantidad        | INT     | Cantidad vendida                             |
| fecha_venta     | DATE    | Fecha de la venta                            |

---

## Inserts de ejemplo

Se insertan algunos clientes, productos y ventas para poblar las tablas y poder probar las funciones.

---

## Funciones PL/pgSQL

### `total_pagado_cliente(cliente_id INT) → NUMERIC`
Devuelve el total de dinero gastado por un cliente.

```sql
SELECT total_pagado_cliente(1);
```

---

### `total_productos_cliente(cliente_id INT) → INT`
Devuelve la cantidad total de productos comprados por un cliente.

---

### `producto_mas_vendido() → VARCHAR`
Devuelve el nombre del producto más vendido (por cantidad).

---

### `facturacion_total() → NUMERIC`
Devuelve la suma total facturada (todas las ventas).

---

### `cliente_compro_producto(cliente_id INT, producto_id INT) → BOOLEAN`
Indica si un cliente compró un producto específico.

---

### `clientes_con_compras() → INT`
Devuelve la cantidad de clientes que realizaron al menos una compra.

---

### `cliente_mas_compras() → VARCHAR`
Devuelve el nombre del cliente que más productos compró (por cantidad total).

---

### `productos_sin_ventas() → TABLE(nombre VARCHAR)`
Lista los productos que nunca se vendieron.

---

### `cantidad_total_vendida(producto_id INT) → INT`
Devuelve la cantidad total vendida de un producto.

---

### `dias_ultima_compra(cliente_id INT) → INT`
Devuelve la cantidad de días desde la última compra de un cliente. Si nunca compró, devuelve -1.

---

### `listar_ventas_cliente(cliente_id INT) → TABLE(producto VARCHAR, cantidad INT, fecha DATE)`
Lista todas las ventas de un cliente, mostrando producto, cantidad y fecha.

---

## Notas y advertencias

- Las funciones usan nombres de tablas y columnas con sufijo `O` (por ejemplo, `clientesO`). Ojo si tenés otras tablas con nombres similares.
- Las referencias en las claves foráneas de la tabla `ventasO` deberían apuntar a `clientesO` y `productosO`, pero en el script están como `clientes` y `productos`. Si no existen esas tablas, tirará error. Cambialo si hace falta.
- Las funciones usan parámetros con el mismo nombre que las columnas, lo que puede causar confusión en algunos motores SQL. Si ves resultados raros, revisá los nombres de variables.

---

