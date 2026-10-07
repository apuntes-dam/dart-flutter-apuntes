# 9.C Buenas prácticas: pool, seguridad, transacciones y DAO

## Pool de conexiones

Abrir una conexión con un servidor de bases de datos es **lento y costoso** (red, autenticación). Un programa que abre y cierra una conexión por cada operación desperdicia mucho tiempo. Un **pool de conexiones** mantiene varias conexiones **ya abiertas** y las **presta** a quien las necesita:

```text
programa ──pide conexión──▶  [ pool: 🔌 🔌 🔌 🔌 ]  ──▶  servidor de base de datos
         ◀──la devuelve───
```

En Dart, los paquetes de cada servidor traen su propia gestión de conexiones (por ejemplo, el paquete `postgres` incluye un `Pool`). Con SQLite, que es un archivo local, basta con **una única conexión compartida** por toda la aplicación (patrón *singleton*).

## Seguridad: la inyección SQL

La **inyección SQL** ocurre cuando se construye una consulta **pegando texto del usuario** dentro del SQL. Si el usuario escribe SQL en lugar de un dato normal, **cambia el significado de la consulta**: puede ver datos que no debe, saltarse un inicio de sesión o borrar tablas. Es una de las vulnerabilidades más graves y frecuentes de las aplicaciones reales.

```dart
import 'package:sqlite3/sqlite3.dart';

void preparar(Database db) {
  db.execute('CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, stock INTEGER NOT NULL)');
  db.execute('''
    INSERT INTO libros (titulo, stock) VALUES
      ('Don Quijote', 3), ('Novelas ejemplares', 2), ('Cien años de soledad', 5), ('El coronel no tiene quien le escriba', 1)''');
}

List<String> buscarInseguro(Database db, String titulo) {
  final sql = "SELECT titulo FROM libros WHERE titulo = '$titulo'"; // ¡NUNCA así!
  return [for (final fila in db.select(sql)) fila['titulo'] as String];
}

List<String> buscarSeguro(Database db, String titulo) {
  final filas = db.select('SELECT titulo FROM libros WHERE titulo = ?', [titulo]);
  return [for (final fila in filas) fila['titulo'] as String];
}

void main() {
  final db = sqlite3.openInMemory();
  preparar(db);
  const maliciosa = "x' OR '1'='1";
  print('[inseguro] Don Quijote -> ${buscarInseguro(db, 'Don Quijote').length} resultado(s)');
  print('[inseguro] $maliciosa -> ${buscarInseguro(db, maliciosa).length} resultado(s)');
  print('[seguro] Don Quijote -> ${buscarSeguro(db, 'Don Quijote').length} resultado(s)');
  print('[seguro] $maliciosa -> ${buscarSeguro(db, maliciosa).length} resultado(s)');
  db.close();
}
```

Salida:

```text
[inseguro] Don Quijote -> 1 resultado(s)
[inseguro] x' OR '1'='1 -> 4 resultado(s)
[seguro] Don Quijote -> 1 resultado(s)
[seguro] x' OR '1'='1 -> 0 resultado(s)
```

La cadena `x' OR '1'='1` hace que la consulta insegura quede como:

```sql
SELECT titulo FROM libros WHERE titulo = 'x' OR '1'='1'
```

Como `'1'='1'` siempre es cierto, la consulta **devuelve todos los libros**. Con una **consulta parametrizada**, esa misma cadena se trata como un **título que no existe**, porque el valor viaja **aparte del SQL** y nunca se interpreta como código.

`db.select('... WHERE titulo = ?', [titulo])`: los valores van en una **lista aparte**.

!!! danger "Regla de oro"
    **Nunca** construyas SQL concatenando o interpolando datos que no controles (todo lo que escribe el usuario, lo que llega de un formulario o de la red). Usa **siempre** parámetros. Un parámetro sirve para **valores**; los nombres de tablas o columnas no se pueden parametrizar: si deben variar, elígelos de una lista fija de opciones permitidas.

## Transacciones

Una **transacción** agrupa varias operaciones para que se ejecuten **como una sola**: o se hacen **todas** o no se hace **ninguna**. Es imprescindible cuando un cambio requiere varios pasos. Prestar un libro, por ejemplo, exige **registrar el préstamo** y **descontar el stock**: si lo primero se hace y lo segundo falla, la base de datos quedaría incoherente.

```dart
import 'package:sqlite3/sqlite3.dart';

class PrestamoFallido implements Exception {
  final String mensaje;
  PrestamoFallido(this.mensaje);
}

void preparar(Database db) {
  db.execute('CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, stock INTEGER NOT NULL)');
  db.execute('CREATE TABLE prestamos (id INTEGER PRIMARY KEY AUTOINCREMENT, id_libro INTEGER NOT NULL, socio TEXT NOT NULL)');
  db.execute('''
    INSERT INTO libros (titulo, stock) VALUES
      ('Don Quijote', 3), ('Novelas ejemplares', 2), ('Cien años de soledad', 5), ('El coronel no tiene quien le escriba', 1)''');
}

int entero(Database db, String sql, [List<Object?> parametros = const []]) =>
    db.select(sql, parametros).first.values.first as int;

/// Registra el préstamo y descuenta el stock como UNA sola operación.
String prestar(Database db, int idLibro, String socio, {bool fallarAMitad = false}) {
  db.execute('BEGIN'); // empieza la transacción
  try {
    if (entero(db, 'SELECT stock FROM libros WHERE id = ?', [idLibro]) < 1) {
      throw PrestamoFallido('no hay stock');
    }
    db.execute('INSERT INTO prestamos (id_libro, socio) VALUES (?, ?)', [idLibro, socio]);
    if (fallarAMitad) {
      throw PrestamoFallido('error simulado a mitad de la operación');
    }
    db.execute('UPDATE libros SET stock = stock - 1 WHERE id = ?', [idLibro]);
    db.execute('COMMIT'); // todo ha ido bien: se confirma
    return 'correcto';
  } on PrestamoFallido catch (e) {
    db.execute('ROLLBACK'); // algo ha fallado: se deshace TODO
    return e.mensaje;
  }
}

void main() {
  final db = sqlite3.openInMemory();
  preparar(db);
  print('préstamo 1: ${prestar(db, 4, 'Ana')}');
  print('préstamo 2: ${prestar(db, 4, 'Luis')}');
  print('préstamo 3: ${prestar(db, 3, 'Eva', fallarAMitad: true)}');
  print('préstamos registrados: ${entero(db, 'SELECT COUNT(*) FROM prestamos')}');
  print('stock del libro 4: ${entero(db, 'SELECT stock FROM libros WHERE id = ?', [4])}');
  print('stock del libro 3: ${entero(db, 'SELECT stock FROM libros WHERE id = ?', [3])}');
  db.close();
}
```

Salida:

```text
préstamo 1: correcto
préstamo 2: no hay stock
préstamo 3: error simulado a mitad de la operación
préstamos registrados: 1
stock del libro 4: 0
stock del libro 3: 5
```

El tercer préstamo falla **a propósito, después de haber insertado el préstamo**. Gracias a la transacción, el `ROLLBACK` **deshace también esa inserción**: al final solo hay **un** préstamo registrado y el stock del libro 3 sigue en 5. Sin transacción, habría un préstamo «fantasma» sin descontar.

Con el paquete `sqlite3` se controla con sentencias: `db.execute('BEGIN')` para empezar, `db.execute('COMMIT')` para confirmar y `db.execute('ROLLBACK')` para deshacer.

Una transacción cumple las propiedades **ACID**:

| Propiedad | Significa |
|---|---|
| **A**tomicidad | Todo o nada |
| **C**onsistencia | Los datos siguen cumpliendo las reglas (claves, restricciones) |
| **I**slamiento | Varias operaciones simultáneas no se estorban |
| **D**urabilidad | Lo confirmado queda guardado aunque falle el equipo |

## El patrón DAO

Si el SQL está **repartido por todo el programa**, cualquier cambio en las tablas obliga a buscarlo en cien sitios, y es imposible probar la lógica sin una base de datos. El patrón **DAO** (*Data Access Object*) lo soluciona: **una clase** concentra **todo** el acceso a los datos de una tabla, y el resto del programa solo habla con ella y con **objetos**, nunca con conexiones ni con SQL.

```text
  Programa  ──▶  Servicio (reglas)  ──▶  DAO (SQL)  ──▶  Base de datos
  objetos Libro        usa objetos            convierte filas ↔ objetos
```

```dart
import 'package:sqlite3/sqlite3.dart';

class Libro {
  final int? id;
  final String titulo;
  final int anio;
  final int stock;
  final int idAutor;

  const Libro(this.id, this.titulo, this.anio, this.stock, this.idAutor);

  Libro conId(int nuevoId) => Libro(nuevoId, titulo, anio, stock, idAutor);
}

/// Único sitio del programa que conoce el SQL y la conexión.
class LibroDao {
  final Database _db;
  LibroDao(this._db);

  static Libro _deFila(Row fila) =>
      Libro(fila['id'] as int, fila['titulo'] as String, fila['anio'] as int, fila['stock'] as int, fila['id_autor'] as int);

  Libro guardar(Libro libro) {
    _db.execute('INSERT INTO libros (titulo, anio, stock, id_autor) VALUES (?, ?, ?, ?)',
        [libro.titulo, libro.anio, libro.stock, libro.idAutor]);
    return libro.conId(_db.lastInsertRowId);
  }

  Libro? buscar(int id) {
    final filas = _db.select('SELECT id, titulo, anio, stock, id_autor FROM libros WHERE id = ?', [id]);
    return filas.isEmpty ? null : _deFila(filas.first);
  }

  List<Libro> todos() =>
      [for (final f in _db.select('SELECT id, titulo, anio, stock, id_autor FROM libros ORDER BY id')) _deFila(f)];

  List<Libro> porAutor(String nombre) {
    const consulta = '''
      SELECT l.id, l.titulo, l.anio, l.stock, l.id_autor FROM libros l
      JOIN autores a ON a.id = l.id_autor WHERE a.nombre = ? ORDER BY l.anio''';
    return [for (final f in _db.select(consulta, [nombre])) _deFila(f)];
  }

  void cambiarStock(int id, int diferencia) {
    _db.execute('UPDATE libros SET stock = stock + ? WHERE id = ?', [diferencia, id]);
  }
}

/// Servicio: usa el DAO y no contiene SQL.
class Catalogo {
  final LibroDao _dao;
  Catalogo(this._dao);

  void prestarUno(int id) {
    final libro = _dao.buscar(id);
    if (libro == null) {
      throw ArgumentError('el libro no existe');
    }
    if (libro.stock < 1) {
      throw StateError('no hay stock');
    }
    _dao.cambiarStock(id, -1);
  }
}

String texto(Libro l) => 'Libro(id=${l.id}, titulo=${l.titulo}, anio=${l.anio}, stock=${l.stock})';

void preparar(Database db) {
  db.execute('CREATE TABLE autores (id INTEGER PRIMARY KEY AUTOINCREMENT, nombre TEXT NOT NULL)');
  db.execute('''
    CREATE TABLE libros (
      id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, anio INTEGER, stock INTEGER NOT NULL,
      id_autor INTEGER NOT NULL REFERENCES autores(id))''');
  db.execute("INSERT INTO autores (nombre) VALUES ('Cervantes'), ('García Márquez'), ('Frank Herbert')");
  db.execute('''
    INSERT INTO libros (titulo, anio, stock, id_autor) VALUES
      ('Don Quijote', 1605, 3, 1), ('Novelas ejemplares', 1613, 2, 1),
      ('Cien años de soledad', 1967, 5, 2), ('El coronel no tiene quien le escriba', 1961, 1, 2)''');
}

void main() {
  final db = sqlite3.openInMemory();
  preparar(db);
  final dao = LibroDao(db);
  final catalogo = Catalogo(dao);

  final nuevo = dao.guardar(const Libro(null, 'Dune', 1965, 2, 3));
  print('guardado: ${texto(nuevo)}');
  print('buscar(5): ${dao.buscar(5)?.titulo}');
  print('buscar(99): ${dao.buscar(99) == null ? 'no existe' : 'existe'}');
  print('del autor Cervantes: ${dao.porAutor('Cervantes').map((l) => l.titulo).join(', ')}');
  catalogo.prestarUno(5);
  print('stock de Dune tras prestar uno: ${dao.buscar(5)?.stock}');
  print('total de libros: ${dao.todos().length}');
  db.close();
}
```

Salida:

```text
guardado: Libro(id=5, titulo=Dune, anio=1965, stock=2)
buscar(5): Dune
buscar(99): no existe
del autor Cervantes: Don Quijote, Novelas ejemplares
stock de Dune tras prestar uno: 1
total de libros: 5
```

Fíjate en el reparto de responsabilidades:

* **`Libro`** es un objeto de datos, sin SQL.
* **`LibroDao`** es el **único** sitio con SQL y con la conexión. Convierte filas en objetos `Libro` y al revés.
* **`Catalogo`** es el servicio: contiene las **reglas** («no se puede prestar sin stock») y usa el DAO, pero **no contiene SQL**.
* Si mañana cambia el motor de base de datos, solo se modifica el DAO.

!!! tip "Relación con SOLID"
    El DAO aplica la **responsabilidad única** (cada clase tiene una sola razón para cambiar) y, si el servicio recibe el DAO por el constructor, la **inversión de dependencias** (ver [6.B](../u06/t-solid-1.md) y [6.C](../u06/t-solid-2.md)): el servicio se podría probar con un DAO falso, sin base de datos.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Concatenar texto del usuario en el SQL | Parámetros `?`, siempre |
| Varias operaciones relacionadas sin transacción | Agruparlas con `BEGIN`/`COMMIT` y deshacer ante cualquier error |
| Olvidar el `ROLLBACK` cuando algo falla | Capturar el error y deshacer siempre |
| SQL esparcido por todo el programa | Una capa DAO |
| Abrir una conexión por cada operación contra un servidor | Un pool, o una conexión compartida si es SQLite |

## Para practicar

Haz los ejercicios de [U9.3 · Pool, seguridad, transacciones y DAO](buenas-practicas.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
