## 🧱 Estructura y diseño de la base de datos

| Caso / Campo típico                                        | Convención recomendada                                                                                                       | Motivo / Buenas prácticas                                                          |
| ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **ID principal**                                           | `id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY`                                                                                 | Identificador único y auto incremental. `UNSIGNED` evita negativos y amplía rango. |
| **Campos obligatorios (usuario, contraseña, email, etc.)** | `NOT NULL`                                                                                                                   | Obliga a completar datos esenciales y evita registros incompletos.                 |
| **Email de usuario**                                       | `VARCHAR(255) UNIQUE NOT NULL`                                                                                               | Evita duplicados y respeta la longitud máxima estándar.                            |
| **Contraseña**                                             | `password VARCHAR(255) NOT NULL`                                                                                             | Soporta hashes (`bcrypt`, `argon2`, etc.). Nunca guardar texto plano.              |
| **Booleanos**                                              | `is_active BOOLEAN DEFAULT TRUE`, `is_admin BOOLEAN DEFAULT FALSE`                                                           | Claros, semánticos y con valores por defecto lógicos.                              |
| **Fechas de creación y actualización**                     | `created_at DATETIME DEFAULT CURRENT_TIMESTAMP`, `updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | Auditoría automática de cambios.                                                   |
| **Campos numéricos sin negativos**                         | `UNSIGNED` (ej: `price DECIMAL(10,2) UNSIGNED`, `stock INT UNSIGNED`)                                                        | Evita valores negativos y amplía el rango positivo.                                |
| **Foreign Keys**                                           | `FOREIGN KEY (campo_id) REFERENCES otra_tabla(id) ON DELETE CASCADE ON UPDATE CASCADE`                                       | Mantiene integridad referencial; elimina dependencias automáticamente.             |
| **Campos de estado o tipo**                                | `ENUM('active','inactive','pending') NOT NULL DEFAULT 'active'` o `TINYINT(1)`                                               | Controla valores válidos y facilita legibilidad.                                   |
| **Campos de texto largos (bio, descripción, comentarios)** | `TEXT` o `LONGTEXT`                                                                                                          | Permite textos amplios sin límite fijo.                                            |
| **Campos opcionales (avatar, teléfono secundario, etc.)**  | `NULL` permitido                                                                                                             | Flexibilidad cuando no todos los usuarios completan esos campos.                   |
| **Campos de borrado lógico**                               | `deleted BOOLEAN DEFAULT FALSE`                                                                                              | Permite mantener registros eliminados sin perder historial.                        |
| **Nombres de tabla y columna**                             | `snake_case`, sin mayúsculas, acentos ni espacios. Ej: `user_profiles`, `order_items`.                                       | Uniformidad y compatibilidad multiplataforma.                                      |
| **Charset / Collation**                                    | `utf8mb4_unicode_ci` o `utf8mb4_general_ci`                                                                                  | Soporta emojis y caracteres internacionales.                                       |
| **Storage Engine**                                         | `ENGINE=InnoDB`                                                                                                              | Soporta transacciones y claves foráneas.                                           |

---

## 🧍‍♂️ 1. Entidades de personas / usuarios

### Campos típicos y por qué se definen así

```sql
name VARCHAR(100) NOT NULL
```

* Siempre `NOT NULL` porque una persona sin nombre no tiene sentido lógico.
* 100 caracteres suele ser suficiente incluso con nombres compuestos.

```sql
surname VARCHAR(100) NOT NULL
```

* Igual criterio.
* No se recomienda unir nombre y apellido en un solo campo (dificulta búsquedas).

```sql
email VARCHAR(255) UNIQUE NOT NULL
```

* `UNIQUE` para evitar duplicados de cuentas.
* `NOT NULL` porque el correo es parte del login o contacto.
* `255` es el límite máximo permitido por estándar.

```sql
password VARCHAR(255) NOT NULL
```

* Espacio suficiente para hashes (`bcrypt`, `argon2`).
* Nunca se guarda texto plano.
* `NOT NULL` porque el usuario debe tener credenciales válidas.

```sql
is_active BOOLEAN DEFAULT TRUE
```

* Permite desactivar un usuario sin borrarlo.
* `DEFAULT TRUE` evita tener que definirlo al crear.

```sql
created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
```

* Auditoría automática, estándar en cualquier tabla que se modifique.

```sql
last_login DATETIME NULL
```

* Se deja `NULL` hasta que el usuario realmente inicie sesión.

```sql
email_verified_at DATETIME NULL
```

* Fecha del momento en que el usuario confirmó su correo.
* `NULL` significa “pendiente de verificación”.

---

## 🏢 2. Clientes / empresas

```sql
company_name VARCHAR(255) NOT NULL
```

* Siempre requerido: sin nombre no hay entidad comercial.

```sql
tax_id VARCHAR(30) UNIQUE NOT NULL
```

* CUIT/CUIL/RUC o similar.
* `UNIQUE` porque es identificador fiscal único.

```sql
contact_email VARCHAR(255) NULL
```

* A veces no tienen correo registrado, por eso `NULL` permitido.

```sql
phone VARCHAR(20) NULL
```

* `VARCHAR` (no numérico) porque puede tener `+`, `-`, espacios.

```sql
address TEXT NULL
```

* Se usa `TEXT` si puede ser largo (calles, referencias, etc.).

```sql
is_client BOOLEAN DEFAULT TRUE
```

* Permite usar la misma tabla para contactos potenciales (`FALSE`) o clientes reales (`TRUE`).

---

## 👩‍🏫 3. Profesores, alumnos o empleados

```sql
employee_number VARCHAR(20) UNIQUE NOT NULL
```

* Código interno o legajo.
* `UNIQUE` para evitar duplicados.

```sql
hire_date DATE NOT NULL
```

* Fecha obligatoria: indica antigüedad laboral.

```sql
birth_date DATE NULL
```

* `NULL` si no se conoce o no es obligatorio.

```sql
salary DECIMAL(10,2) UNSIGNED NOT NULL
```

* `UNSIGNED` porque no tiene sentido un salario negativo.
* `DECIMAL` para evitar errores de redondeo.

```sql
is_full_time BOOLEAN DEFAULT FALSE
```

* Permite distinguir contratos o tipo de trabajo.

```sql
department_id INT UNSIGNED,
FOREIGN KEY (department_id) REFERENCES departments(id) ON DELETE SET NULL
```

* Si se elimina el departamento, no queremos borrar al empleado, solo dejarlo sin asignación.

---

## 📦 4. Productos / inventario

```sql
name VARCHAR(150) NOT NULL
```

* `NOT NULL` porque un producto sin nombre no tiene sentido.

```sql
sku VARCHAR(50) UNIQUE NOT NULL
```

* Código único por producto.

```sql
price DECIMAL(10,2) UNSIGNED NOT NULL DEFAULT 0.00
```

* `UNSIGNED` porque no existen precios negativos.
* `DEFAULT 0.00` útil para evitar errores si no se define.

```sql
stock INT UNSIGNED NOT NULL DEFAULT 0
```

* Nunca negativo; `UNSIGNED`.
* `DEFAULT 0` inicializa correctamente el inventario.

```sql
is_available BOOLEAN DEFAULT TRUE
```

* Permite marcar un producto como descontinuado sin borrarlo.

```sql
category_id INT UNSIGNED NOT NULL,
FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE CASCADE
```

* Si se borra una categoría, se eliminan sus productos.
* Ideal cuando los productos dependen completamente de esa categoría.

---

## 💳 5. Ventas / pedidos

```sql
order_number VARCHAR(50) UNIQUE NOT NULL
```

* Número de pedido único, útil para trazabilidad.

```sql
user_id INT UNSIGNED NOT NULL,
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
```

* Si se elimina un usuario, también sus pedidos (dependencia directa).

```sql
total_amount DECIMAL(10,2) UNSIGNED NOT NULL
```

* Monto total, no negativo.

```sql
status ENUM('pending', 'paid', 'shipped', 'cancelled') DEFAULT 'pending'
```

* Controla estados válidos.
* ENUM es eficiente y legible para valores fijos.

```sql
paid_at DATETIME NULL
```

* `NULL` hasta que el pago se confirme.

---

## 🔐 6. Roles y permisos

```sql
name VARCHAR(50) UNIQUE NOT NULL
```

* Nombre único: “admin”, “editor”, “user”.

```sql
description TEXT NULL
```

* Descripción opcional.

```sql
is_default BOOLEAN DEFAULT FALSE
```

* Indica si es el rol asignado por defecto a nuevos usuarios.

```sql
created_at DATETIME DEFAULT CURRENT_TIMESTAMP
```

* Auditoría de creación.

Tabla intermedia (usuarios ↔ roles):

```sql
user_id INT UNSIGNED NOT NULL,
role_id INT UNSIGNED NOT NULL,
PRIMARY KEY (user_id, role_id),
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
```

* Clave compuesta para evitar duplicados.
* `CASCADE` para mantener integridad.

---

## 🧾 7. Auditoría y lógica de negocio

```sql
deleted BOOLEAN DEFAULT FALSE
```

* “Soft delete”: permite marcar registros como eliminados sin perder el historial.

```sql
deleted_at DATETIME NULL
```

* Guarda cuándo fue eliminado lógicamente.

```sql
created_by INT UNSIGNED NULL,
updated_by INT UNSIGNED NULL,
FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE SET NULL
```

* Guarda quién creó o modificó cada registro.

```sql
version INT UNSIGNED DEFAULT 1
```

* Para sistemas con control de versiones de datos (documentos, contratos, etc.).

---

## 🧩 8. Otras prácticas generales

* **Usar `InnoDB`** siempre: soporta transacciones y foreign keys.
* **Definir `utf8mb4`** en todas las tablas: soporte total de Unicode y emojis.
* **Usar `snake_case`** para nombres (`user_profiles`, no `UserProfiles`).
* **Evitar `NULL` en campos booleanos**: usar `DEFAULT 0` o `1`.
* **Prefijos claros**: `user_id`, `order_id`, `product_id` → más legible en joins.
* **Evitar tipos `FLOAT` o `DOUBLE`** para dinero o precisión contable.
* **Evitar `TEXT` para campos que se filtran frecuentemente** (no indexables fácilmente).
* **Evitar `ENUM` si el conjunto de valores puede crecer dinámicamente**, usar tabla de referencia en su lugar.
* **Siempre indexar foreign keys** para performance.
