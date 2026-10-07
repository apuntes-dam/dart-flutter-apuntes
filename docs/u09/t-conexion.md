# 9.A Conectar y crear tablas

## Qué es una base de datos relacional

Una **base de datos relacional** guarda la información en **tablas**: cada tabla tiene **columnas** (los datos que se guardan) y **filas** (cada elemento guardado). Las tablas se **relacionan** entre sí mediante claves, y todo se maneja con un lenguaje común, **SQL**.

| Concepto | Qué es | Ejemplo |
|---|---|---|
| **Tabla** | Un conjunto de datos del mismo tipo | `libros` |
| **Fila** (registro) | Un elemento guardado | el libro «Don Quijote» |
| **Columna** (campo) | Un dato de cada fila, con un tipo | `titulo`, `anio`, `stock` |
| **Clave primaria** (*primary key*) | Columna que **identifica de forma única** cada fila | `id` |
| **Clave foránea** (*foreign key*) | Columna que **apunta a la clave primaria de otra tabla** | `libros.id_autor` → `autores.id` |

En los ejemplos de esta unidad hay tres tablas relacionadas: un **autor** tiene muchos **libros** y un **libro** puede tener muchos **préstamos**.

```text
autores (id, nombre)  1 ──── N  libros (id, titulo, anio, stock, id_autor)  1 ──── N  prestamos (id, id_libro, socio)
```

## Motores de bases de datos

| Motor | Cómo funciona | Cuándo se usa |
|---|---|---|
| **SQLite** | Un **archivo** (o la memoria); no hay servidor ni instalación | Aprender, apps móviles y de escritorio, pruebas |
| **H2**, HSQLDB | Base de datos **embebida en Java** (archivo o memoria) | Pruebas y programas Java sencillos |
| **MySQL**, **PostgreSQL** | **Servidor** al que se conecta por red con usuario y contraseña | Aplicaciones reales con muchos usuarios |

!!! note "Qué motor usan estos ejemplos"
    Todos los ejemplos usan **SQLite**, porque no necesita instalar nada. Los ejercicios proponen **H2** para Java y Kotlin: el código JDBC es el mismo, solo cambian la **dependencia** (`com.h2database:h2`) y la **URL** (`jdbc:h2:./data/tienda`). Yo he comprobado los ejemplos con SQLite, no con H2. Las sentencias también varían un poco entre motores: por ejemplo, la columna que se numera sola es `AUTOINCREMENT` en SQLite y `AUTO_INCREMENT` en MySQL y H2.

## Conectar, crear las tablas e insertar datos

Se usa el paquete **`sqlite3`** de pub.dev (`dart pub add sqlite3`; estos ejemplos se han probado con la versión 3.7). `sqlite3.open('biblioteca.db')` abre (o crea) la base de datos en un archivo y `sqlite3.openInMemory()` la crea solo en memoria. Las sentencias se lanzan con `db.execute(sql, [parametros])` (sin resultado) o `db.select(sql, [parametros])` (con filas), y la conexión se cierra con **`db.close()`**. Los errores de SQL son una **`SqliteException`**.

```dart
import 'dart:io';

import 'package:sqlite3/sqlite3.dart';

void main() {
  final db = sqlite3.open('biblioteca.db');
  try {
    print('conectado a la base de datos');
    db.execute('CREATE TABLE autores (id INTEGER PRIMARY KEY AUTOINCREMENT, nombre TEXT NOT NULL)');
    db.execute('''
      CREATE TABLE libros (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        titulo TEXT NOT NULL,
        anio INTEGER,
        stock INTEGER NOT NULL,
        id_autor INTEGER NOT NULL REFERENCES autores(id))''');
    db.execute('''
      CREATE TABLE prestamos (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        id_libro INTEGER NOT NULL REFERENCES libros(id),
        socio TEXT NOT NULL)''');
    print('tablas creadas');

    final autores = ['Cervantes', 'García Márquez'];
    final insertarAutor = db.prepare('INSERT INTO autores (nombre) VALUES (?)');
    for (final nombre in autores) {
      insertarAutor.execute([nombre]);
    }
    insertarAutor.close();
    print('autores insertados: ${autores.length}');

    final libros = [
      ['Don Quijote', 1605, 3, 1],
      ['Novelas ejemplares', 1613, 2, 1],
      ['Cien años de soledad', 1967, 5, 2],
      ['El coronel no tiene quien le escriba', 1961, 1, 2],
    ];
    final insertarLibro = db.prepare('INSERT INTO libros (titulo, anio, stock, id_autor) VALUES (?, ?, ?, ?)');
    for (final libro in libros) {
      insertarLibro.execute(libro);
    }
    insertarLibro.close();
    print('libros insertados: ${libros.length}');

    final total = db.select('SELECT COUNT(*) AS total FROM libros').first['total'];
    print('libros en la base de datos: $total');

    try {
      db.execute("INSERT INTO editoriales (nombre) VALUES ('X')");
    } on SqliteException {
      print('Error controlado: la sentencia no es válida');
    }
  } on SqliteException catch (e) {
    print('Error de base de datos: ${e.message}');
  } finally {
    db.close();
    print('conexión cerrada');
  }

  File('biblioteca.db').deleteSync();
  print('archivo biblioteca.db borrado');
}
```

Salida:

```text
conectado a la base de datos
tablas creadas
autores insertados: 2
libros insertados: 4
libros en la base de datos: 4
Error controlado: la sentencia no es válida
conexión cerrada
archivo biblioteca.db borrado
```

Qué hace cada paso:

1. **Conecta** con la base de datos (el archivo `biblioteca.db`, que se crea si no existe).
2. **Crea las tres tablas** con `CREATE TABLE`, indicando para cada columna su tipo y sus restricciones.
3. **Inserta datos** con `INSERT` y **parámetros** (`?`): los valores no se pegan dentro del texto SQL, se pasan aparte. Esto es lo que se explica en [9.C](t-buenas-practicas.md) y es **obligatorio** cuando el dato viene del usuario.
4. **Cuenta** las filas con una consulta.
5. **Provoca un error a propósito** (insertar en una tabla que no existe) y lo **captura**: el programa no se detiene y muestra un mensaje claro.
6. **Cierra la conexión** (`db.close()` en el bloque `finally`) **siempre**, haya habido error o no. Solo después se puede borrar el archivo: con la conexión abierta, Windows no permite borrarlo.

## Crear tablas: tipos y restricciones

```sql
CREATE TABLE libros (
  id       INTEGER PRIMARY KEY AUTOINCREMENT,   -- identifica cada fila; se numera solo
  titulo   TEXT    NOT NULL,                    -- obligatorio
  anio     INTEGER,                             -- puede quedar vacío (NULL)
  stock    INTEGER NOT NULL,
  id_autor INTEGER NOT NULL REFERENCES autores(id)   -- clave foránea
);
```

| Restricción | Significa |
|---|---|
| `PRIMARY KEY` | Identifica la fila; no se repite ni puede ser nula |
| `NOT NULL` | La columna no puede quedar vacía |
| `UNIQUE` | No puede haber dos filas con el mismo valor (por ejemplo, un correo) |
| `REFERENCES tabla(columna)` | Clave foránea: el valor **debe existir** en la otra tabla |
| `DEFAULT valor` | Valor que se usa si no se indica ninguno |

| Tipo en SQLite | Para qué | Equivalente habitual en otros motores |
|---|---|---|
| `INTEGER` | Números enteros | `INT`, `BIGINT` |
| `TEXT` | Texto | `VARCHAR(n)`, `TEXT` |
| `REAL` | Números decimales | `DOUBLE`, `FLOAT` |
| `BLOB` | Datos binarios | `BLOB`, `BYTEA` |

!!! tip "Dinero y decimales"
    Para importes, otros motores ofrecen `DECIMAL(10,2)`, que guarda decimales **exactos**. Con `REAL` (coma flotante) pueden aparecer errores de redondeo, como en `0.1 + 0.2`. Una alternativa muy usada es guardar el importe **en céntimos**, como entero.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| No cerrar la conexión | `finally` / `try-with-resources` / `use` |
| Pegar el valor dentro del SQL | Usar parámetros `?` |
| Olvidar confirmar los cambios (en Python, `commit`) | Confirmar al terminar o usar una transacción ([9.C](t-buenas-practicas.md)) |
| Crear una tabla que ya existe | Crear las tablas una vez, o usar `CREATE TABLE IF NOT EXISTS` |
| Insertar con claves foráneas que no existen | Insertar primero la tabla «padre» (los autores) y luego la «hija» (los libros) |

## Para practicar

Haz los ejercicios de [U9.1 · Conexión y creación de tablas](conexion.md). [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) compara cómo se escribe en cada lenguaje.
