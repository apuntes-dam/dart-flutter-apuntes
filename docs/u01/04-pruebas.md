# 1.4 Pruebas con `package:test`

Una **prueba unitaria** comprueba automáticamente que una función devuelve lo esperado.

## Preparar el proyecto

```bash
dart create calculadora
cd calculadora
dart pub add dev:test
```

## La función

```dart
// lib/calculadora.dart
int suma(int a, int b) => a + b;

double dividir(int a, int b) {
  if (b == 0) throw ArgumentError('b no puede ser 0');
  return a / b;
}
```

## La prueba

```dart
// test/calculadora_test.dart
import 'package:calculadora/calculadora.dart';
import 'package:test/test.dart';

void main() {
  group('suma', () {
    test('suma dos positivos', () => expect(suma(2, 3), 5));
    test('suma con cero', () => expect(suma(5, 0), 5));
  });

  test('dividir por cero lanza error', () {
    expect(() => dividir(1, 0), throwsArgumentError);
  });
}
```

```bash
dart test
```

!!! tip "Patrón AAA"
    **A**rrange (preparar), **A**ct (ejecutar), **A**ssert (comprobar).
