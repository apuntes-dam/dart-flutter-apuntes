# A4.A Qué es una prueba y cómo se escribe

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5. La [unidad 1](../../u01/04-pruebas.md) ya te enseñó una primera prueba; aquí aprendes a escribirlas con método.

Una **prueba automática** es un programa corto que ejecuta tu código con unos datos **y comprueba el resultado por ti**. Lo que compruebas a mano una vez, la prueba lo comprueba **todas las veces**, en segundos, cada vez que cambias algo. Si rompes algo sin querer, lo sabes **al momento** y no dentro de una semana.

## La receta: preparar, actuar, comprobar

Casi todas las pruebas siguen tres pasos (*Arrange, Act, Assert*):

| Paso | Qué haces | En el ejemplo |
|---|---|---|
| **Preparar** | Creas los objetos y los datos | Un carrito nuevo |
| **Actuar** | Llamas al código que se prueba | `agregar('tarta', 1800, 2)` |
| **Comprobar** | Dices qué esperas | El total es `3600` |

Una buena prueba **comprueba una sola cosa**, tiene un **nombre que dice lo que debe ocurrir** y **no depende de otra prueba**.

## El marco de pruebas de Dart

Dart usa el paquete **`test`** (`dart pub add --dev test`). Las pruebas van en la carpeta `test/`, en archivos cuyo nombre acaba en `_test.dart`. Cada `test('descripción', () { ... })` es una prueba y `expect(valor, esperado)` es la comprobación. Se ejecutan con **`dart test`**. El código se importa con `import 'package:carrito_app/carrito.dart'`, donde `carrito_app` es el `name:` del `pubspec.yaml`.

## Un ejemplo: un carrito de la compra

Primero, el código que se prueba. Los precios van en **céntimos** (enteros) para evitar errores de decimales:

**`carrito.dart`**

```dart
// El código que se prueba: un carrito de la compra.
// Los precios van en céntimos (números enteros) para evitar errores de redondeo con decimales.
class Carrito {
  final List<({String producto, int precio, int cantidad})> _lineas = [];

  void agregar(String producto, int precio, int cantidad) {
    if (cantidad <= 0) throw ArgumentError('la cantidad debe ser mayor que cero');
    if (precio < 0) throw ArgumentError('el precio no puede ser negativo');
    _lineas.add((producto: producto, precio: precio, cantidad: cantidad));
  }

  int get total => _lineas.fold(0, (suma, l) => suma + l.precio * l.cantidad);
  int get unidades => _lineas.fold(0, (suma, l) => suma + l.cantidad);
  bool get vacio => _lineas.isEmpty;

  /// El total después de aplicar un descuento de 0 a 100 por ciento.
  int conDescuento(int porcentaje) {
    if (porcentaje < 0 || porcentaje > 100) {
      throw ArgumentError('el porcentaje debe estar entre 0 y 100');
    }
    return total - total * porcentaje ~/ 100;
  }
}
```

Y sus tres primeras pruebas:

**`carrito_test.dart`**

```dart
import 'package:carrito_app/carrito.dart';
import 'package:test/test.dart';

void main() {
  late Carrito carrito;

  setUp(() {
    carrito = Carrito(); // un carrito nuevo antes de CADA prueba: así no dependen unas de otras
  });

  test('un carrito nuevo está vacío y su total es 0', () {
    expect(carrito.vacio, isTrue);
    expect(carrito.total, 0);
  });

  test('agregar un producto suma su importe', () {
    carrito.agregar('tarta', 1800, 2);
    expect(carrito.total, 3600);
    expect(carrito.unidades, 2);
  });

  test('con varios productos, el total es la suma de los importes', () {
    carrito.agregar('tarta', 1800, 2);
    carrito.agregar('galleta', 100, 12);
    expect(carrito.total, 4800);
  });
}
```

`setUp(() { ... })` se ejecuta **antes de cada prueba**. Por eso el carrito se vuelve a crear: ninguna prueba hereda lo que hizo la anterior.

Al ejecutar las pruebas:

```text
+0: un carrito nuevo está vacío y su total es 0
+1: agregar un producto suma su importe
+2: con varios productos, el total es la suma de los importes
+3: All tests passed!
```

`+3` es el número de pruebas que han pasado hasta ese momento, y el final dice `All tests passed!`.

## Cuando una prueba falla

Para ver un fallo **de verdad**, rompí el código a propósito: cambié `precio × cantidad` por `precio + cantidad` en el cálculo del total. Esto es lo que se ve:

```text
+1 -1: agregar un producto suma su importe [E]
  Expected: <3600>
    Actual: <1802>
  test\carrito_test.dart 18:5  main.<fn>
```

Un buen mensaje de fallo dice **qué prueba falló**, **qué se esperaba** (`3600`) y **qué se obtuvo** (`1802`). Con eso suele bastar para encontrar el error sin depurar.

!!! tip "Una prueba que nunca ha fallado no demuestra nada"
    Cuando escribas una prueba, **rómpela una vez a propósito** (cambia el resultado esperado o el código) y comprueba que falla. Así sabes que de verdad está mirando algo.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Pruebas que dependen unas de otras (el orden importa) | Prepara de cero antes de cada prueba |
| Nombres como `test1`, `test2` | Describe el comportamiento: «agregar un producto suma su importe» |
| Comprobar muchas cosas distintas en una sola prueba | Una idea por prueba: si falla, sabes qué falló |
| Copiar en la prueba el mismo cálculo que hace el código | Escribe el resultado esperado **a mano** (`3600`), no `precio * cantidad` |

## Para practicar

Los ejercicios [A4.1 y A4.4](ejercicios.md) piden escribir una función con sus pruebas y una preparación previa. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
