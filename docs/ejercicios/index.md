# Ejercicios de Dart

Colección de **73 ejercicios** de programación adaptados a Dart: del primer programa a las excepciones y la depuración. Los enunciados están redactados de nuevo a partir de los de la asignatura de Programación (UD 1 y UD 2).

| Bloque | Ejercicios | Tema |
|---|---|---|
| [P1.2 · Primeros programas](p1-2.md) | 32 | Entrada/salida, variables, operadores, cadenas |
| [P2.1 · Sentencias condicionales](p2-1.md) | 10 | `if`, `else`, selección múltiple |
| [P2.2 · Sentencias iterativas y saltos](p2-2.md) | 25 | bucles `for` y `while` |
| [P2.3 · Captura de excepciones](p2-3.md) | 5 | `try`, excepciones propias |
| [P2.4 · Depurar programas](p2-4.md) | 1 | Algoritmo de la burbuja y depurador |

!!! tip "Cómo trabajar"
    Para cada ejercicio, anota primero **entradas, proceso y salidas**. Escribe el programa en un archivo independiente, pruébalo con valores normales, cero y negativos, y solo entonces consulta la solución modelo (si la hay).

!!! info "Soluciones bloqueadas"
    Hay una solución probada por cada tipo de ejercicio, pero está **bloqueada**: solo se ve el comienzo como ejemplo de cómo es la solución. Está cifrada y el administrador la desbloquea con el botón **🔒 Admin** (abajo a la derecha).

## Herramientas útiles en Dart

| Necesitas... | Usa... |
|---|---|
| Leer una línea | `stdin.readLineSync()` (de `dart:io`) |
| Texto a número | `int.parse(...)`, `double.parse(...)`, `int.tryParse(...)` |
| Redondear al mostrar | `numero.toStringAsFixed(2)` |
| Potencia y raíz | `pow(x, 2)`, `sqrt(x)` (de `dart:math`) |
| Aleatorios | `Random().nextInt(n)` (de `dart:math`) |
| Mayúsculas / minúsculas | `toUpperCase()`, `toLowerCase()` |
| Cortar y unir texto | `split(',')`, `substring(a, b)`, `join(', ')` |
| Rellenar con ceros | `'7'.padLeft(3, '0')` |
| Lanzar una excepción | `throw ...` / `rethrow` |

Para ejecutar cada ejercicio: `dart run ejercicio.dart`.
