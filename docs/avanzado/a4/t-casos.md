# A4.B Errores y casos límite

El código casi nunca falla con los datos «normales»: falla en los **bordes** (el primero, el último, el cero, el vacío) y con los **datos inválidos**. Esas son las pruebas que más valen.

## Probar que algo falla

También hay que comprobar que el código **rechaza** lo incorrecto, con el error y el mensaje adecuados.

`expect(() => f(), throwsArgumentError)` comprueba que el código **lanza** ese error: se pasa una **función** (con `() =>`) para que `expect` la ejecute y capture el error. Para mirar el mensaje, `throwsA(isA<ArgumentError>().having((e) => e.message, 'mensaje', '...'))`.

## Casos límite

Un **caso límite** es un valor en la frontera entre dos comportamientos. Si un descuento va de `0` a `100`, prueba `0` y `100` (los extremos válidos) y `-1` y `101` (justo fuera). Los errores de «uno de más o de menos» (`<` en vez de `<=`) viven ahí.

| Qué pruebas | Valores típicos |
|---|---|
| Números con un rango | El mínimo, el máximo, justo debajo y justo encima |
| Colecciones y texto | **Vacío**, un solo elemento, muchos |
| Búsquedas | El primero, el último, uno que **no está** |
| Divisiones y porcentajes | Cero, uno, el límite |

## Varios casos con una tabla

Cuando la comprobación es la misma y solo cambian los datos, se escribe **una prueba** que recorre una **tabla de casos** (entrada y resultado esperado):

Un mapa constante y un `forEach` dentro de **una** prueba; el parámetro `reason:` dice qué caso falló.

## El ejemplo

Las pruebas del carrito: dos para los errores, una con la tabla de descuentos y otra para el porcentaje fuera de rango.

**`carrito_casos_test.dart`**

```dart
import 'package:carrito_app/carrito.dart';
import 'package:test/test.dart';

void main() {
  late Carrito carrito;

  setUp(() {
    carrito = Carrito();
  });

  test('una cantidad cero o negativa lanza un error con un mensaje claro', () {
    expect(
      () => carrito.agregar('tarta', 1800, 0),
      throwsA(isA<ArgumentError>().having((e) => e.message, 'mensaje', 'la cantidad debe ser mayor que cero')),
    );
    expect(() => carrito.agregar('tarta', 1800, -1), throwsArgumentError);
  });

  test('un precio negativo lanza un error', () {
    expect(() => carrito.agregar('tarta', -5, 1), throwsArgumentError);
  });

  test('descuentos de 0, 10, 50 y 100 por ciento (casos límite incluidos)', () {
    carrito.agregar('tarta', 1800, 2);
    carrito.agregar('galleta', 100, 12);
    const casos = {0: 4800, 10: 4320, 50: 2400, 100: 0}; // porcentaje -> total esperado
    casos.forEach((porcentaje, esperado) {
      expect(carrito.conDescuento(porcentaje), esperado, reason: 'con $porcentaje %');
    });
  });

  test('un porcentaje fuera de 0 a 100 lanza un error', () {
    expect(() => carrito.conDescuento(101), throwsArgumentError);
    expect(() => carrito.conDescuento(-1), throwsArgumentError);
  });
}
```

Al ejecutar las pruebas:

```text
+0: una cantidad cero o negativa lanza un error con un mensaje claro
+1: un precio negativo lanza un error
+2: descuentos de 0, 10, 50 y 100 por ciento (casos límite incluidos)
+3: un porcentaje fuera de 0 a 100 lanza un error
+4: All tests passed!
```

Los descuentos esperados se escribieron **a mano** (`4800 → 4320` con un `10 %`): si copiaras la fórmula del código en la prueba, repetirías también sus errores.

!!! warning "Cobertura no es calidad"
    Un programa de pruebas puede **ejecutar** todas las líneas del código y aun así no comprobar casi nada. Mide cuántos **casos importantes** tienes cubiertos (bordes, errores, datos raros), no solo cuántas líneas pasan por las pruebas.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Probar solo el caso feliz | Añade siempre al menos un borde y un error |
| Comprobar que «lanza algún error» sin mirar cuál | Comprueba el **tipo** y, si importa, el **mensaje** |
| Un bucle de casos donde el primer fallo oculta a los demás | Usa el mecanismo del marco para indicar el caso (`reason`, `subTest`...) |
| Escribir las pruebas copiando la implementación | Calcula el resultado esperado a mano o con otra vía |

## Para practicar

Los ejercicios [A4.2, A4.3 y A4.5](ejercicios.md) piden pruebas de casos normales, de errores y de límites con una tabla. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
