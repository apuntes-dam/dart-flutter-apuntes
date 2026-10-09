# 7.E Ficheros con dart:io, bytes y bloques

En [7.B](t-archivos.md) y [7.C](t-texto.md) ya usaste `File`, `Directory` y funciones como `readAsStringSync`. Aquí se completa el cuadro con lo que suele pedirse en el módulo de Acceso a Datos:

* Las **excepciones concretas** de cada operación y cómo reaccionar a ellas.
* Los **ficheros binarios**: se guardan **bytes**, no letras.
* **Leer y escribir por bloques**: en vez de cargar todo el archivo de golpe, se trabaja con un trozo cada vez.

Dart no trae una clase `Path` en el SDK (existe un paquete `path`, que hay que instalar): las rutas son **textos** y se unen escribiendo la barra `/`, que funciona en todos los sistemas. Todo esto está en `dart:io`.

Todos los ejemplos de esta página se han **ejecutado** con Dart en una carpeta vacía, en Windows.

## File y Directory: operaciones y errores

| Quiero… | Se escribe |
|---|---|
| Saber si existe | `File(r).existsSync()` · `Directory(r).existsSync()` |
| Saber si es archivo o carpeta | `FileSystemEntity.isFileSync(r)` · `FileSystemEntity.isDirectorySync(r)` |
| Nombre y carpeta padre | `File(r).uri.pathSegments.last` · `File(r).parent.path` |
| Ruta absoluta | `File(r).absolute.path` |
| Tamaño en bytes | `File(r).lengthSync()` |
| Crear una carpeta (y las que falten) | `Directory(r).createSync(recursive: true)` |
| Crear un archivo vacío | `File(r).createSync(exclusive: true)` |
| Copiar | `File(r).copySync(destino)` |
| Mover o cambiar de nombre | `File(r).renameSync(destino)` |
| Borrar un archivo | `File(r).deleteSync()` |
| Borrar una carpeta vacía (o con todo dentro) | `Directory(r).deleteSync()` (`recursive: true`) |

Un recorrido completo, con `try`, `on … catch` y un `finally` que limpia lo que se haya creado:

```dart
import 'dart:io';

void main() {
  final carpeta = Directory('datos');
  final nota = File('datos/nota.txt');
  try {
    carpeta.createSync(recursive: true);
    nota.createSync(exclusive: true);
    nota.writeAsStringSync('hola');
    print('Ruta absoluta: ${nota.absolute.path}');
    print('Nombre: ${nota.uri.pathSegments.last} | Padre: ${nota.parent.path}');
    print('Es archivo: ${FileSystemEntity.isFileSync(nota.path)} | Es carpeta: ${FileSystemEntity.isDirectorySync(nota.path)}');
    print('Tamaño: ${nota.lengthSync()} bytes');
    final copia = nota.copySync('datos/copia.txt');
    final movido = copia.renameSync('datos/final.txt');
    print('Existe copia.txt: ${copia.existsSync()} | Existe final.txt: ${movido.existsSync()}');
  } on FileSystemException catch (e) {
    print('Error: ${e.runtimeType} - ${e.message}');
  } finally {
    final final_ = File('datos/final.txt');
    if (final_.existsSync()) final_.deleteSync();
    if (nota.existsSync()) nota.deleteSync();
    if (carpeta.existsSync()) carpeta.deleteSync();
    print('Limpieza hecha: existe la carpeta = ${carpeta.existsSync()}');
  }
}
```

**Salida:**

```text
Ruta absoluta: C:\Users\ana\proyecto\datos/nota.txt
Nombre: nota.txt | Padre: datos
Es archivo: true | Es carpeta: false
Tamaño: 4 bytes
Existe copia.txt: false | Existe final.txt: true
Limpieza hecha: existe la carpeta = false
```

El `finally` se ejecuta **siempre**, haya habido error o no. Por eso es el sitio para borrar lo que el programa creó. Fíjate en que Dart conserva la barra que escribiste (`datos/nota.txt`), aunque en Windows la ruta absoluta empiece por `C:\`.

### Qué pasa cuando algo falla

Los errores de `dart:io` son excepciones de la familia **`FileSystemException`**. Mira cuáles saltan y, sobre todo, **cuáles no**:

```dart
import 'dart:io';

void intentar(String texto, Object? Function() accion) {
  try {
    print('$texto -> ${accion()}');
  } on FileSystemException catch (e) {
    print('$texto -> ${e.runtimeType}: ${e.message}');
  }
}

void main() {
  final carpeta = Directory('datos');
  final nota = File('datos/nota.txt');
  intentar('crear carpeta', () { carpeta.createSync(); return 'ok'; });
  intentar('crear la carpeta otra vez', () { carpeta.createSync(); return 'ok'; });
  intentar('crear archivo (exclusive)', () { nota.createSync(exclusive: true); return 'ok'; });
  intentar('crear el archivo otra vez (exclusive)', () { nota.createSync(exclusive: true); return 'ok'; });
  intentar('crear el archivo otra vez (sin exclusive)', () { nota.createSync(); return 'ok'; });
  intentar('tamaño de uno que no existe', () => File('datos/no.txt').lengthSync());
  intentar('copiar', () => nota.copySync('datos/copia.txt').path);
  intentar('copiar sobre uno que existe', () => nota.copySync('datos/copia.txt').path);
  intentar('copiar uno que no existe', () => File('datos/no.txt').copySync('datos/x.txt').path);
  intentar('borrar uno que no existe', () { File('datos/no.txt').deleteSync(); return 'ok'; });
  intentar('borrar una carpeta con cosas dentro', () { carpeta.deleteSync(); return 'ok'; });
  intentar('crear subcarpeta sin la carpeta padre', () { Directory('datos/a/b').createSync(); return 'ok'; });
  intentar('crear subcarpeta con recursive', () { Directory('datos/a/b').createSync(recursive: true); return 'ok'; });
  intentar('renombrar sobre uno que existe', () => nota.renameSync('datos/copia.txt').path);
  intentar('leer uno que no existe', () => File('datos/no.txt').readAsStringSync());
}
```

**Salida:**

```text
crear carpeta -> ok
crear la carpeta otra vez -> ok
crear archivo (exclusive) -> ok
crear el archivo otra vez (exclusive) -> PathExistsException: Cannot create file
crear el archivo otra vez (sin exclusive) -> ok
tamaño de uno que no existe -> PathNotFoundException: Cannot retrieve length of file
copiar -> datos/copia.txt
copiar sobre uno que existe -> datos/copia.txt
copiar uno que no existe -> PathNotFoundException: Cannot copy file to 'datos/x.txt'
borrar uno que no existe -> PathNotFoundException: Cannot delete file
borrar una carpeta con cosas dentro -> FileSystemException: Deletion failed
crear subcarpeta sin la carpeta padre -> PathNotFoundException: Creation failed
crear subcarpeta con recursive -> ok
renombrar sobre uno que existe -> datos/copia.txt
leer uno que no existe -> PathNotFoundException: Cannot open file
```

| Excepción | Cuándo se produce |
|---|---|
| `PathExistsException` | Crear un archivo con `exclusive: true` donde ya hay uno |
| `PathNotFoundException` | La ruta no existe: tamaño, copia, borrado, lectura... o crear una carpeta cuando falta la carpeta padre |
| `FileSystemException` | La general: las dos anteriores son casos suyos. Es la que sale al borrar una carpeta con cosas dentro |

Se capturan de la más concreta a la más general: `on PathExistsException`, `on PathNotFoundException` y, al final, `on FileSystemException`. En el mensaje, `e.message` explica la operación y `e.path` dice qué ruta falló.

!!! warning "Lo que NO da error"
    `createSync()` no falla si la carpeta ya existe, `copySync` **sobrescribe** el destino sin avisar y `renameSync` también. Si no quieres perder datos, comprueba antes con `existsSync()`.

## Ficheros binarios

En un fichero binario cada elemento es un **byte**, un número entre 0 y 255. Se abre con `openSync(mode: FileMode.write)` (escribir) o `openSync()` (leer), que devuelve un `RandomAccessFile`. En Dart no hay un bloque que lo cierre solo: hay que llamar a **`closeSync()`**, y lo seguro es hacerlo en un **`finally`**.

* `salida.writeByteSync(numero)` escribe **un** byte.
* `entrada.readByteSync()` lee un byte (un `int` de 0 a 255). Cuando no queda nada, devuelve **`-1`**.

```dart
import 'dart:io';

void main() {
  final fichero = File('datos.bin');

  final salida = fichero.openSync(mode: FileMode.write);
  try {
    for (final numero in [72, 111, 108, 97]) {
      salida.writeByteSync(numero);
    }
  } finally {
    salida.closeSync();
  }
  print('Tamaño: ${fichero.lengthSync()} bytes');

  final entrada = fichero.openSync();
  try {
    var dato = entrada.readByteSync();
    while (dato != -1) {
      print('Leído $dato = ${String.fromCharCode(dato)}');
      dato = entrada.readByteSync();
    }
  } finally {
    entrada.closeSync();
  }
}
```

**Salida:**

```text
Tamaño: 4 bytes
Leído 72 = H
Leído 111 = o
Leído 108 = l
Leído 97 = a
```

Para archivos pequeños hay atajos: `writeAsBytesSync(lista)` y `readAsBytesSync()`.

!!! warning "Un valor mayor que 255 no da error"
    Los bytes de Dart van de 0 a 255 (nunca salen negativos), pero si escribes un número mayor, Dart **se queda con los 8 bits de abajo** sin avisar: el 300 se guarda como `300 − 256 = 44`.

```dart
import 'dart:io';

void main() {
  final fichero = File('datos.bin');

  fichero.writeAsBytesSync([200]);
  print('readAsBytesSync(): ${fichero.readAsBytesSync()}');

  final salida = fichero.openSync(mode: FileMode.write);
  salida.writeByteSync(300);        // no cabe en un byte
  salida.closeSync();
  print('Después de escribir 300: ${fichero.readAsBytesSync()}');
}
```

**Salida:**

```text
readAsBytesSync(): [200]
Después de escribir 300: [44]
```

### Leer y escribir por bloques

Leer byte a byte es lento con archivos grandes. Lo habitual es usar un **buffer** (un `Uint8List`) y leer un bloque de golpe:

* `entrada.readIntoSync(buffer)` rellena el buffer y **devuelve cuántos bytes ha leído** (0 cuando ya no queda nada).
* `salida.writeFromSync(buffer, 0, leidos)` escribe **solo** los `leidos` primeros bytes del buffer.

```dart
import 'dart:io';
import 'dart:typed_data';

void main() {
  final origen = File('origen.bin');
  origen.writeAsBytesSync(List.generate(25, (i) => 65 + i));   // 25 bytes de prueba

  final copia = File('copia.bin');
  final entrada = origen.openSync();
  final salida = copia.openSync(mode: FileMode.write);
  try {
    final buffer = Uint8List(10);
    var leidos = entrada.readIntoSync(buffer);
    while (leidos > 0) {
      print('Bloque de $leidos bytes');
      salida.writeFromSync(buffer, 0, leidos);
      leidos = entrada.readIntoSync(buffer);
    }
  } finally {
    entrada.closeSync();
    salida.closeSync();
  }
  print('Origen: ${origen.lengthSync()} bytes | Copia: ${copia.lengthSync()} bytes');
}
```

**Salida:**

```text
Bloque de 10 bytes
Bloque de 10 bytes
Bloque de 5 bytes
Origen: 25 bytes | Copia: 25 bytes
```

El último bloque trae solo 5 bytes, aunque el buffer sea de 10. Si se escribe el buffer entero, se copia basura:

```dart
import 'dart:io';
import 'dart:typed_data';

void main() {
  final origen = File('origen.bin');
  origen.writeAsBytesSync(List.generate(25, (i) => 65 + i));

  final mala = File('mala.bin');
  final entrada = origen.openSync();
  final salida = mala.openSync(mode: FileMode.write);
  try {
    final buffer = Uint8List(10);
    var leidos = entrada.readIntoSync(buffer);
    while (leidos > 0) {
      salida.writeFromSync(buffer);        // ¡escribe los 10 bytes aunque se hayan leído menos!
      leidos = entrada.readIntoSync(buffer);
    }
  } finally {
    entrada.closeSync();
    salida.closeSync();
  }
  print('Origen: ${origen.lengthSync()} bytes | Copia mal hecha: ${mala.lengthSync()} bytes');
}
```

**Salida:**

```text
Origen: 25 bytes | Copia mal hecha: 30 bytes
```

!!! warning "Escribe solo lo que has leído"
    `writeFromSync(buffer)` escribe **todo** el buffer. En el último bloque conserva los bytes del bloque anterior, así que la copia sale más grande y estropeada. Usa siempre `writeFromSync(buffer, 0, leidos)`.

## Ficheros de texto: caracteres, bytes y bloques

Un archivo de texto también son bytes, y Dart usa **UTF-8** por defecto. En UTF-8 las letras sin tilde ocupan 1 byte, pero la `ñ`, las vocales con tilde o el `€` ocupan 2 o 3. Por eso **caracteres y bytes no coinciden**. Además, `length` en Dart cuenta unidades de 16 bits (UTF-16), no «letras»: un emoji cuenta como 2.

```dart
import 'dart:convert';
import 'dart:io';

void main() {
  const texto = 'Año 2026: 5 € ñandú\nsegunda línea\n';
  final fichero = File('texto.txt');
  fichero.writeAsStringSync(texto);

  print('length: ${texto.length}');
  print('runes.length: ${texto.runes.length}');
  print('Bytes en UTF-8: ${utf8.encode(texto).length}');
  print('Bytes en el disco: ${fichero.lengthSync()}');

  const carita = 'a😀';
  print('"a😀": length ${carita.length}, runes ${carita.runes.length}, bytes ${utf8.encode(carita).length}');
}
```

**Salida:**

```text
length: 34
runes.length: 34
Bytes en UTF-8: 40
Bytes en el disco: 40
"a😀": length 3, runes 2, bytes 5
```

`runes.length` cuenta los caracteres Unicode de verdad. Para saber cuánto ocupa en el disco hay que mirar `utf8.encode(texto).length` o `lengthSync()`.

### Leer texto por bloques

Dart **no tiene una función que lea «n caracteres»**. Hay dos caminos:

1. **Leer bloques de bytes** y decodificarlos con `utf8.decode`. Es peligroso: un bloque puede cortar un carácter de varios bytes por la mitad, y entonces `decode` lanza una `FormatException`:

```dart
import 'dart:convert';
import 'dart:io';
import 'dart:typed_data';

void main() {
  final fichero = File('texto.txt');
  fichero.writeAsStringSync('Año 5 €');             // 10 bytes: ñ ocupa 2 y € ocupa 3

  final lector = fichero.openSync();
  try {
    final buffer = Uint8List(2);                    // bloques de 2 BYTES
    var leidos = lector.readIntoSync(buffer);
    var n = 1;
    while (leidos > 0) {
      final trozo = buffer.sublist(0, leidos);
      String texto;
      try {
        texto = utf8.decode(trozo);
      } on FormatException {
        texto = '(un carácter partido por la mitad)';
      }
      print('Bloque $n ($leidos bytes) $trozo -> $texto');
      leidos = lector.readIntoSync(buffer);
      n++;
    }
  } finally {
    lector.closeSync();
  }
}
```

**Salida:**

```text
Bloque 1 (2 bytes) [65, 195] -> (un carácter partido por la mitad)
Bloque 2 (2 bytes) [177, 111] -> (un carácter partido por la mitad)
Bloque 3 (2 bytes) [32, 53] ->  5
Bloque 4 (2 bytes) [32, 226] -> (un carácter partido por la mitad)
Bloque 5 (2 bytes) [130, 172] -> (un carácter partido por la mitad)
```

2. **Dejar que Dart lea y decodifique por ti** con `openRead().transform(utf8.decoder)`. Es un `Stream` que entrega **trozos de texto ya decodificados**, sin cortar caracteres, y se recorre con `await for` dentro de una función `async`. El tamaño del trozo lo decide Dart (en torno a 64 KB), no tú:

```dart
import 'dart:convert';
import 'dart:io';

Future<void> main() async {
  final fichero = File('grande.txt');
  fichero.writeAsStringSync('línea con ñ\n' * 20000);
  print('Tamaño: ${fichero.lengthSync()} bytes');

  var trozos = 0;
  var caracteres = 0;
  await for (final trozo in fichero.openRead().transform(utf8.decoder)) {
    trozos++;
    caracteres += trozo.length;
    print('Trozo $trozos: ${trozo.length} caracteres');
  }
  print('En total: $trozos trozos y $caracteres caracteres');
}
```

**Salida:**

```text
Tamaño: 280000 bytes
Trozo 1: 56173 caracteres
Trozo 2: 56174 caracteres
Trozo 3: 56174 caracteres
Trozo 4: 56174 caracteres
Trozo 5: 15305 caracteres
En total: 5 trozos y 240000 caracteres
```

Un trozo puede acabar **en medio de una línea**: el resto de esa línea llega en el trozo siguiente. Por eso, si cuentas líneas o palabras por trozos, no se puede suponer que cada trozo traiga líneas completas.

Para **escribir** por bloques sí se decide el tamaño: se cortan trozos del texto con `substring(inicio, fin)` y se escriben con `writeStringSync`:

```dart
import 'dart:io';

void main() {
  const letras = 'ABCDEFGHIJKLMNOPQRSTU';             // 21 caracteres
  final destino = File('letras.txt');
  final escritor = destino.openSync(mode: FileMode.write);
  var escrituras = 0;
  try {
    for (var inicio = 0; inicio < letras.length; inicio += 8) {
      final fin = (inicio + 8 < letras.length) ? inicio + 8 : letras.length;   // el último trozo es más corto
      escritor.writeStringSync(letras.substring(inicio, fin));
      escrituras++;
    }
  } finally {
    escritor.closeSync();
  }
  print('Escrituras: $escrituras');
  print('Contenido: ${destino.readAsStringSync()}');
}
```

**Salida:**

```text
Escrituras: 3
Contenido: ABCDEFGHIJKLMNOPQRSTU
```

!!! note "Y si solo quieres las líneas"
    Para recorrer un archivo línea a línea no hace falta leer por bloques: `readAsLinesSync()` o `openRead().transform(utf8.decoder).transform(const LineSplitter())` (ver [7.C](t-texto.md)). Los bloques se usan cuando el archivo es grande o cuando te piden trabajar con un tamaño fijo.

## Errores frecuentes

* **Olvidar `closeSync()`** (o no ponerlo en un `finally`): el archivo se queda abierto y puede no guardarse todo lo escrito.
* **Esperar un error donde Dart no lo da**: crear una carpeta que existe, copiar o renombrar sobre un archivo existente, o escribir un byte mayor que 255.
* **Escribir el buffer entero** en vez de `writeFromSync(buffer, 0, leidos)`.
* **Decodificar bloques de bytes sin cuidado**: un carácter partido lanza `FormatException`.
* **Confundir `length` con el número de caracteres**: usa `runes.length`.
* **Olvidar `await for`** (y `async`) al leer con `openRead()`: devuelve un `Stream`, no un texto.

## Para practicar

Los ejercicios [7.16 a 7.21](path.md) usan todo lo de esta página.
