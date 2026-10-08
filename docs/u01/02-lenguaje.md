# 1.2 Práctica con el lenguaje

## Estructura de un programa Dart

```dart
// 1. Importaciones
import 'dart:math';

// 2. Funciones y clases
int cuadrado(int n) => n * n;

// 3. Punto de entrada
void main() {
  print(cuadrado(5)); // 25
  print(sqrt(16));    // 4.0
}
```

Todo programa empieza en `main()`. Las **sentencias** terminan en `;` y los **bloques** van entre `{ }`.

## Variables

```dart
var nombre = 'Ana';        // tipo inferido: String
int edad = 20;             // tipo explícito
double altura = 1.68;
bool activo = true;
String? apodo;             // puede ser null (null safety)
```

!!! note "Null safety"
    Por defecto una variable **no** puede valer `null`. Para permitirlo se añade `?` al tipo. Para decir "sé que no es null" se usa `!`.

## Constantes: `final` y `const`

| Palabra | Cuándo se asigna | Ejemplo |
|---|---|---|
| `final` | una vez, en ejecución | `final ahora = DateTime.now();` |
| `const` | en compilación | `const dosPi = 3.14159 * 2;` |

## Literales

`42` (int), `3.14` (double), `'hola'` o `"hola"` (String), `true` (bool), `[1, 2]` (List), `{1, 2}` (Set), `{'a': 1}` (Map).

## Operadores

```dart
void main() {
  print(7 + 2);   // 9
  print(7 / 2);   // 3.5  (siempre double)
  print(7 ~/ 2);  // 3    (división entera)
  print(7 % 2);   // 1
  print(3 > 2 && 2 > 1); // true
  String? x;
  print(x ?? 'sin valor'); // sin valor
}
```

Grupos: aritméticos (`+ - * / ~/ %`), relacionales (`== != < > <= >=`), lógicos (`&& || !`), asignación (`= += -=`), y los propios de Dart: `??` (valor por defecto) y `?.` (acceso seguro).

## Entrada y salida

```dart
import 'dart:io';

void main() {
  stdout.write('¿Cómo te llamas? ');
  final nombre = stdin.readLineSync() ?? 'anónimo';
  print('Hola, $nombre');   // interpolación
}
```


### Leer números y otros tipos

`stdin.readLineSync()` devuelve siempre **texto**, y puede ser `null` si ya no hay datos (por eso se escribe `?? '0'`). Para guardar un número hay que **convertir** ese texto:

| Quiero leer… | Cómo se escribe | Tipo que queda |
|---|---|---|
| Un texto | `final texto = stdin.readLineSync() ?? '';` | `String` |
| Un entero | `final n = int.parse(stdin.readLineSync() ?? '0');` | `int` |
| Un decimal | `var coste = double.parse(stdin.readLineSync() ?? '0');` | `double` |
| Un sí/no | `final acepta = (stdin.readLineSync() ?? '').trim().toLowerCase() == 's';` | `bool` |
| Varios números en una línea | `final v = (stdin.readLineSync() ?? '').split(' ').map(int.parse).toList();` | `List<int>` |

Los cuatro primeros, en un programa (con `var` o `final` Dart deduce el tipo; `runtimeType` lo comprueba):

```dart
import 'dart:io';

void main() {
  print('Edad:');
  final edad = int.parse(stdin.readLineSync() ?? '0');        // int
  print('Altura en metros:');
  final altura = double.parse(stdin.readLineSync() ?? '0');   // double
  print('Nombre:');
  final nombre = stdin.readLineSync() ?? 'anónimo';           // String
  print('¿Aceptas las condiciones? (s/n):');
  final acepta = (stdin.readLineSync() ?? '').trim().toLowerCase() == 's';   // bool

  print('$nombre tiene $edad años y mide $altura m');
  print('El año que viene tendrá ${edad + 1}');
  print('Acepta: $acepta');
  print('Tipos: ${edad.runtimeType}, ${altura.runtimeType}, ${nombre.runtimeType}, ${acepta.runtimeType}');
}
```

Con esta entrada: `17`, `1.75`, `Ana` y `s` (una por línea).

```text
Edad:
Altura en metros:
Nombre:
¿Aceptas las condiciones? (s/n):
Ana tiene 17 años y mide 1.75 m
El año que viene tendrá 18
Acepta: true
Tipos: int, double, String, bool
```

!!! warning "Un error muy típico"
    El `??` solo elige **entre dos textos**; no convierte nada. Por eso esta línea no compila:

    ```dart
    int a = stdin.readLineSync() ?? '0';     // ¡error!
    ```

    ```text
    Error: A value of type 'String' can't be assigned to a variable of type 'int'.
    ```

    Hay que convertir **después** de decidir el valor por defecto: `int.parse(stdin.readLineSync() ?? '0')`.

!!! note "El `?? '0'` solo cubre «no hay datos»"
    Si el usuario escribe algo que **no es un número** (`abc`) o un decimal con **coma** (`1,75`), `int.parse` y `double.parse` fallan igualmente. Para ese caso, `tryParse` y más detalles están en «Más a fondo».

??? note "Más a fondo: `null`, datos mal escritos, coma decimal, varios valores…"
    **¿Qué es `String?` y para qué sirven `!` y `??`?**

    `readLineSync()` devuelve `null` cuando **ya no hay más datos**: el usuario pulsa **Ctrl+D** (Linux y Mac) o **Ctrl+Z** y Enter (Windows), o la entrada viene de un archivo que se acabó. Como `int.parse` no acepta `null`, Dart obliga a decidir qué hacer:

    | Opción | Qué significa | Si no hay datos |
    |---|---|---|
    | `stdin.readLineSync()!` | «Estoy seguro de que hay una línea» | **Error** al ejecutar |
    | `stdin.readLineSync() ?? '0'` | «Si no hay, usa este valor» | Usa el valor por defecto |
    | `if (linea == null) { ... }` | Tratar el caso a mano | Lo que tú decidas |

    Con `!`, si no hay datos el programa se detiene. Este es el mensaje real de Dart con la entrada vacía:

    ```text
    Unhandled exception:
    Null check operator used on a null value
    ```

    **Cuando lo que escribe el usuario no es un número**

    `int.parse` **lanza una excepción** si el texto no es un entero. Sin controlarla, el programa se detiene:

    ```dart
    import 'dart:io';

    void main() {
      final n = int.parse(stdin.readLineSync() ?? '0');
      print('El doble es ${n * 2}');
    }
    ```

    Con esta entrada: `abc`.

    ```text
    Unhandled exception:
    FormatException: Invalid radix-10 number (at character 1)
    abc
    ^
    ```

    `int.tryParse` devuelve **`null` en lugar de lanzar la excepción**, y así se puede **volver a preguntar**:

    ```dart
    import 'dart:io';

    int pedirEntero(String mensaje) {
      while (true) {
        print(mensaje);
        final linea = stdin.readLineSync();
        if (linea == null) throw StateError('No hay más datos de entrada');
        final n = int.tryParse(linea);                         // null si no es un entero (ya ignora los espacios de los lados)
        if (n != null) return n;
        print('  Eso no es un número entero. Prueba otra vez.');
      }
    }

    void main() {
      final n = pedirEntero('Escribe un número entero:');
      print('Has escrito el $n');
    }
    ```

    Con esta entrada: `abc`, `4.5` y `  12  ` (con espacios), una por línea. Ni `abc` ni `4.5` son enteros; `parse` y `tryParse` ya ignoran los espacios de los lados.

    ```text
    Escribe un número entero:
      Eso no es un número entero. Prueba otra vez.
    Escribe un número entero:
      Eso no es un número entero. Prueba otra vez.
    Escribe un número entero:
    Has escrito el 12
    ```

    También se trata el `null` (`if (linea == null) throw ...`): si la entrada se acabara, el bucle **no se quedaría preguntando para siempre**. Otra forma corta, con valor por defecto: `var coste = double.tryParse(stdin.readLineSync() ?? '') ?? 0;`.

    **Decimales: el punto, no la coma**

    `double.parse` y `double.tryParse` entienden **el punto** como separador decimal. Con `1,75` hay que cambiar la coma por un punto antes de convertir:

    ```dart
    import 'dart:io';

    void main() {
      final texto = stdin.readLineSync()!;
      print(double.tryParse(texto));                       // null: con coma no es un double
      print(double.tryParse(texto.replaceAll(',', '.')));  // 1.75: se cambia la coma por el punto
    }
    ```

    Con esta entrada: `1,75`.

    ```text
    null
    1.75
    ```

    **Varios valores en una misma línea**

    Con `split` se parte la línea y con `map(int.parse)` se convierte cada trozo. `RegExp(r'\s+')` significa «uno o más espacios seguidos»:

    ```dart
    import 'dart:io';

    void main() {
      final numeros = (stdin.readLineSync() ?? '')
          .trim()
          .split(RegExp(r'\s+'))        // uno o más espacios seguidos
          .map(int.parse)
          .toList();
      print(numeros);
      print('Suma: ${numeros.reduce((a, b) => a + b)}');
    }
    ```

    Con esta entrada: `3 4  5` (con dos espacios entre el 4 y el 5).

    ```text
    [3, 4, 5]
    Suma: 12
    ```

    **Leer hasta que se acabe la entrada**

    Como `readLineSync()` devuelve `null` al final, se puede leer línea a línea **hasta que no haya más**, ignorando lo que no sea un número:

    ```dart
    import 'dart:io';

    void main() {
      var suma = 0;
      String? linea;
      while ((linea = stdin.readLineSync()) != null) {        // null cuando no hay más líneas
        final n = int.tryParse(linea!.trim());
        if (n == null) {
          print('Ignorada: "$linea"');
          continue;
        }
        suma += n;
      }
      print('Suma: $suma');
    }
    ```

    Con esta entrada: `10`, `20`, `hola` y `5`, una por línea.

    ```text
    Ignorada: "hola"
    Suma: 35
    ```

    **Booleanos: `true`/`false` frente a «s/n»**

    Desde Dart 3 existe `bool.parse`, pero solo entiende las palabras **`true` y `false`** (con `caseSensitive: false` ignora las mayúsculas). Para preguntas como «¿Aceptas? (s/n)» lo habitual es **comparar el texto con una letra**, como en el primer ejemplo.

    ```dart
    import 'dart:io';

    void main() {
      final a = bool.parse(stdin.readLineSync()!, caseSensitive: false);
      final b = bool.parse(stdin.readLineSync()!);
      print('$a $b');
      print(bool.tryParse('si'));                             // null: solo reconoce true y false
    }
    ```

    Con esta entrada: `TRUE` y `false`.

    ```text
    true false
    null
    ```

    Las salidas de esta sección se han obtenido **ejecutando cada programa con Dart 3.13 y la entrada indicada**.

## Comentarios

```dart
// Comentario de una línea
/* Comentario
   de varias líneas */
/// Comentario de documentación (aparece en el IDE)
```

## Crear un proyecto y usar el IDE

```bash
dart create mi_proyecto
cd mi_proyecto
dart run
```

Se pueden usar VS Code (extensiones *Dart* y *Flutter*), Android Studio o IntelliJ.
