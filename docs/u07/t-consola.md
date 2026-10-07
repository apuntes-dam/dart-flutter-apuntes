# 7.A Consola: entrada y salida estándar

Cuando un programa se ejecuta en una terminal, el sistema le da **tres canales** de texto, llamados **flujos estándar**:

| Flujo | Nombre | Para qué sirve | Por defecto |
|---|---|---|---|
| `stdin` | Entrada estándar | Lo que el programa **recibe** | El teclado |
| `stdout` | Salida estándar | Los **resultados** normales | La pantalla |
| `stderr` | Salida de error | Los **avisos y errores** | La pantalla |

Los dos de salida van a la pantalla, pero son **canales distintos**, y esa diferencia permite separar los resultados de los mensajes de error.

## Mostrar datos con formato

```dart
import 'dart:io';

void main() {
  final equipos = [('Águilas', 45, 12), ('Lobos', 41, 5), ('Tigres', 38, -2)];

  print('${'Equipo'.padRight(12)}${'Pts'.padLeft(5)}${'Dif'.padLeft(6)}');
  print('-' * 23);
  var total = 0;
  for (final (nombre, puntos, diferencia) in equipos) {
    final signo = diferencia >= 0 ? '+' : '';
    print('${nombre.padRight(12)}${'$puntos'.padLeft(5)}${'$signo$diferencia'.padLeft(6)}');
    total += puntos;
  }
  print('Media de puntos: ${(total / equipos.length).toStringAsFixed(2)}');

  stderr.writeln('Aviso: datos de ejemplo, no oficiales');
  print('Fin del informe');
}
```

Salida:

```text
Equipo        Pts   Dif
-----------------------
Águilas        45   +12
Lobos          41    +5
Tigres         38    -2
Media de puntos: 41.33
Fin del informe
```

Qué conviene aprender de este ejemplo:

* **Columnas alineadas.** Se reserva un **ancho fijo** para cada columna: el texto a la izquierda y los números a la derecha. Sin eso, las columnas quedan descuadradas.
* **Decimales fijos.** La media sale con dos decimales aunque el cálculo dé muchos más.
* **El aviso va por otro canal.** Verás que la línea «Aviso: datos de ejemplo, no oficiales» **no** aparece en el recuadro de arriba: se escribe en `stderr`, no en `stdout`. En una terminal sí se vería.

| Necesito... | En Dart |
|---|---|
| Alinear a la izquierda | `'Ana'.padRight(12)` |
| Alinear a la derecha | `'45'.padLeft(5)` |
| Decimales fijos | `media.toStringAsFixed(2)` |
| Repetir un texto | `'-' * 23` |
| Mostrar sin salto de línea | `stdout.write('texto')` |
| Con signo | se añade a mano: `d >= 0 ? '+$d' : '$d'` |

`toStringAsFixed` siempre usa el punto como separador decimal, sea cual sea la configuración del equipo.

## Salida estándar y salida de error

Separarlas sirve para que un programa pueda **guardar los resultados en un archivo sin mezclarlos con los errores**. En una terminal, el operador `>` redirige la salida normal y `2>` la de error:

```bash
programa > resultados.txt 2> errores.txt
```

Con el ejemplo anterior, `resultados.txt` contendría la tabla y la media y `errores.txt` solo el aviso:

```bash
dart run cons1.dart > resultados.txt 2> errores.txt
```

Para **dar entrada desde un archivo** en lugar del teclado se usa `<` (`programa < entrada.txt`), y para encadenar programas, la **tubería** `|`.

## Leer datos

Se lee con **`stdin.readLineSync()`** (de `dart:io`), que devuelve un `String?`: es **`null`** cuando ya no queda entrada (*fin de archivo*), por eso hay que contemplarlo (`?? ''`). Para escribir sin saltar de línea se usa `stdout.write`, y para la salida de error, `stderr.writeln`.

```dart
import 'dart:io';

void main() {
  print('Escribe líneas (una vacía para terminar):');
  var lineas = 0;
  var palabras = 0;
  while (true) {
    final linea = stdin.readLineSync();
    if (linea == null || linea.isEmpty) {
      break;
    }
    lineas++;
    palabras += linea.split(' ').where((p) => p.isNotEmpty).length;
  }
  print('Líneas: $lineas');
  print('Palabras: $palabras');
}
```

Este programa lee líneas **hasta encontrar una vacía** (o hasta que se acabe la entrada). Si se le escribe `hola mundo`, `esto es una prueba` y una línea vacía, muestra:

```text
Escribe líneas (una vacía para terminar):
Líneas: 2
Palabras: 6
```

Una entrada del usuario es **siempre texto**: si necesitas un número hay que convertirlo y **prever que falle** (ver el patrón de «pedir hasta que sea válido» en [2.3 Excepciones](../u02/02-excepciones.md)).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Columnas descuadradas | Fijar un ancho para cada columna y alinear números a la derecha |
| Mezclar resultados y errores en la misma salida | Escribir los avisos en `stderr` |
| Dar por hecho que siempre hay una línea más que leer | Comprobar el fin de la entrada (`null`, `hasNextLine`, `EOFError`) |
| Usar el número leído sin convertirlo ni validarlo | Convertir dentro de un `try` y volver a pedir |
| Decimales con coma en unas máquinas y con punto en otras | Fijar la configuración regional al formatear (en Java y Kotlin) |

## Para practicar

Haz los ejercicios de [U7.1 · Consola](consola.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
