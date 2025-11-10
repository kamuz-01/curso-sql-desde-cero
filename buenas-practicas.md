## 🧱 Estructura y diseño de la base de datos

### 1. Usar una clave primaria simple y estable

Siempre crear una columna `id` como **PRIMARY KEY**.

Tipo: `INT UNSIGNED AUTO_INCREMENT`.

**Ejemplo:**

```sql
id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY
```

Nunca uses datos reales (como DNI o email) como clave primaria.
Son cambiables y pueden romper relaciones.

---

### 2. Nombres consistentes y claros

Usar **snake_case** (minúsculas y guiones bajos): `user_account`, `order_item`.

Tablas en singular o plural, pero consistente en todo el proyecto.

**Ejemplo:** `user`, `order`, `product` (singular).

Columnas con prefijo si es necesario:

**Ejemplo:** `user_id`, `order_id` en tablas que hacen referencia a `user` o `order`.

---

### 3. Tipos de datos adecuados

* `INT` o `BIGINT` para IDs.
* `VARCHAR(n)` con tamaño razonable, ej. `VARCHAR(255)` para textos cortos.
* `TEXT` o `LONGTEXT` solo si es realmente necesario (no indexables fácilmente).
* `DECIMAL(10,2)` para dinero o números con precisión fija.
* `DATETIME` o `TIMESTAMP` para fechas.
* Evitá `ENUM` si pensás que los valores pueden cambiar con el tiempo (usá una tabla de referencia).

---

### 4. Relaciones y llaves foráneas

Siempre definir las **FK (Foreign Keys)** para mantener la integridad referencial.

**Ejemplo:**

```sql
FOREIGN KEY (user_id) REFERENCES user(id)
```

Agregá índices en las columnas de FK para mejorar rendimiento.

Usar `ON DELETE CASCADE` o `ON DELETE SET NULL` cuando sea apropiado.

---

### 5. Convenciones de NULL

Evitá `NULL` si no es necesario.
En muchos casos, conviene usar valores por defecto (`0`, `''`, `FALSE`).

Si usás `NULL`, siempre considerá cómo afectará tus consultas (`IS NULL` vs `=`).

---

### 6. Optimización básica

* Evitá `SELECT *`, pedí solo las columnas necesarias.
* Usá `LIMIT` en pruebas o consultas de control.
* Usá `JOIN` correctamente (y no subconsultas anidadas innecesarias).
