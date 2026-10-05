# 1.3 Tipos de datos

Dart es de **tipado estático** con inferencia: el tipo se conoce al compilar.

## Tipos simples

| Tipo | Ejemplo | Nota |
|---|---|---|
| `int` | `42` | entero |
| `double` | `3.14` | decimal |
| `num` | `5` o `5.5` | padre de `int` y `double` |
| `String` | `'hola'` | inmutable |
| `bool` | `true` | solo `true`/`false` |

## Cadenas

```dart
void main() {
  final s = 'Dart';
  print(s.length);                 // 4
  print(s.toUpperCase());          // DART
  print('Hola, $s y ${s.length}'); // interpolación
}
```

## Colecciones

```dart
void main() {
  final lista = <int>[3, 1, 2];        // List: ordenada, admite repetidos
  final conjunto = <int>{1, 1, 2};     // Set: sin repetidos -> {1, 2}
  final mapa = <String, int>{'a': 1};  // Map: clave -> valor

  lista.add(4);
  print(lista);           // [3, 1, 2, 4]
  print(conjunto.length); // 2
  print(mapa['a']);       // 1
}
```

## `dynamic` y `Object`

`dynamic` desactiva las comprobaciones de tipo; se evita salvo necesidad. `Object?` es el supertipo de todo.

## Conversiones

```dart
void main() {
  print(int.parse('42'));      // 42
  print(double.parse('3.5'));  // 3.5
  print(int.tryParse('abc'));  // null (no lanza error)
  print(42.toString());        // 42
  print(3.9.toInt());          // 3 (trunca)
  print(3.toDouble());         // 3.0

  num n = 5;                   // un int es un num
  print(n);
}
```

!!! warning "Dart no convierte solo"
    `double d = 5;` funciona porque `5` es un literal que se interpreta como double, pero `int i = 2.5;` es un error de compilación. Para pasar de `double` a `int` hay que usar `toInt()`, `round()`, `floor()` o `ceil()`.
