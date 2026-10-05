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
