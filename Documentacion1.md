## Documentación: Función libros_entre_anios

### Descripción

La función libros_entre_anios permite obtener todos los libros cuya fecha de publicación (año_publicacion) esté comprendida entre dos años dados (inclusive). Es ideal para filtrar libros publicados en un rango de años específico, por ejemplo, para reportes, estadísticas o búsquedas avanzadas.

---

### Definición

sql
create or replace function libros_entre_anios(
    anio_inicio INT, anio_fin INT
)
returns setof libros as $$
begin
    return query select * from libros
    where extract(year from año_publicacion) between anio_inicio and anio_fin;
end;
$$ language plpgsql;


---

### Parámetros

- *anio_inicio* (INT): Año inicial del rango (inclusive).
- *anio_fin* (INT): Año final del rango (inclusive).

---

### Retorno

- Devuelve un conjunto de filas (SETOF libros) correspondientes a la tabla libros, es decir, todos los campos de los libros publicados entre los años indicados.

---

### Ejemplo de uso

sql
select libros_entre_anios(2022, 2024);

Esto retorna todos los libros publicados entre 2022 y 2024, ambos inclusive.

---
