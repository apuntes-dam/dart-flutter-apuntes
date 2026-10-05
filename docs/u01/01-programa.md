# 1.1 Un programa

Un **programa** es una secuencia de instrucciones que resuelve un problema, y el **algoritmo** son los pasos, en orden, que van de los datos de entrada al resultado.

!!! info "La teoría general está aparte"
    Qué es un algoritmo, el ciclo de desarrollo y el **pseudocódigo** son iguales en todos los lenguajes, así que están en una sola página: [Fundamentos: algoritmos y pseudocódigo](https://apuntes-dam.github.io/apuntes-lenguajes/fundamentos/). Aquí solo ves cómo se traduce a Dart.

## Del pseudocódigo a Dart

El algoritmo del área de un rectángulo, en pseudocódigo:

```text
ALGORITMO areaRectangulo
  LEER base
  LEER altura
  area <- base * altura
  ESCRIBIR "Área:", area
FIN
```

y su traducción a Dart:

```dart
import 'dart:io';

void main() {
  stdout.write('Base: ');
  final base = double.parse(stdin.readLineSync()!);
  stdout.write('Altura: ');
  final altura = double.parse(stdin.readLineSync()!);

  final area = base * altura;
  print('Área: $area');
}
```

| Pseudocódigo | Dart |
|---|---|
| `LEER x` | `stdin.readLineSync()` (de `dart:io`; devuelve texto, se convierte con `double.parse(...)` / `int.parse(...)`) |
| `x <- expresión` | `x = expresión;` |
| `ESCRIBIR x` | `print(x)` |
| `SI ... SINO ... FIN SI` | `if` / `else` (ver [1.2](02-lenguaje.md)) |
| `PARA` / `MIENTRAS` | `for` / `while` (ver [1.6](06-bucles.md)) |

!!! warning "Errores típicos en Dart"
    Mezclar tipos sin convertir (`stdin` da texto) y usar `!` sobre algo que puede ser nulo. Los errores generales de diseño (olvidar un caso, bucles que no terminan…) están en [Fundamentos](https://apuntes-dam.github.io/apuntes-lenguajes/fundamentos/).
