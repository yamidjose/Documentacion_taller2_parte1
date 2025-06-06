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
# Documentación de funciones PL/pgSQL

## 1. Función: compras_clientes

*Definición:*
sql
create or replace function compras_clientes(idcliente int)
returns table (productos_comprados varchar(100)) as $$
begin
  return query
  SELECT p.nombre
  FROM productos p
  JOIN detalle_ventas dv ON p.id_producto = dv.id_producto
  JOIN ventas v ON v.id_venta = dv.id_venta
  WHERE v.id_cliente = idcliente;
END;
$$ LANGUAGE plpgsql;


### Descripción

Esta función retorna una tabla con los nombres de los productos comprados por un cliente específico. Es útil para obtener el historial de compras de un cliente de manera sencilla.

### Parámetros

- *idcliente* (int): ID del cliente del cual se quieren obtener los productos comprados.

### Retorno

- Una tabla con una columna: productos_comprados (varchar(100)), que contiene los nombres de los productos.

### Ejemplo de uso

sql
select * from compras_clientes(1);

Esto devuelve todos los productos que compró el cliente con ID 1.

---

## 2. Función: eliminar_venta

*Definición:*
sql
create or replace function eliminar_venta(idventa int)
returns text as $$
begin
  update productos p
  set stock = stock + dv.cantidad
  from detalle_ventas dv
  where p.id_producto = dv.id_producto
    and dv.id_venta = idventa;

  delete from detalle_ventas
  where id_venta = idventa;

  delete from ventas
  where id_venta = idventa;

  return 'venta eliminada y stock actualizado correctamente';
end;
$$ language plpgsql;


### Descripción

Esta función elimina una venta y actualiza el stock de los productos involucrados. Primero, devuelve el stock de los productos vendidos en esa venta, luego elimina los registros de detalle y finalmente la venta principal.

### Parámetros

- *idventa* (int): ID de la venta que se desea eliminar.

### Retorno

- Un mensaje de texto (text) indicando que la venta fue eliminada y el stock actualizado correctamente.

### Ejemplo de uso

sql
select * from eliminar_venta(2);

Esto elimina la venta con ID 2, actualiza el stock de los productos involucrados y devuelve un mensaje de confirmación.

---

## Notas adicionales

- Ambas funciones están escritas en PL/pgSQL y deben ejecutarse en un entorno PostgreSQL.
- Es importante tener en cuenta las restricciones de integridad referencial en las tablas para evitar errores al eliminar ventas.
- Si usás estas funciones en producción, asegurate de tener backups y considerar transacciones para evitar inconsistencias si algo falla a mitad de camino.

---

