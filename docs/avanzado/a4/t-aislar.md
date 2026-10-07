# A4.C Aislar lo que no controlas

Una prueba tiene que dar **siempre el mismo resultado**. Pero hay código que depende de cosas que **no controlas**: la hora, el azar, una base de datos, una conexión a internet, un archivo. Si tu código llama a esas cosas directamente, sus pruebas **pasarán o fallarán según el día o la hora**.

## La solución: pedir las dependencias desde fuera

En vez de que la tienda pregunte la hora al sistema, **recibe un reloj** al crearse. Es la **inyección de dependencias** (la «D» de [SOLID](../../u06/t-solid-2.md)). En el programa real se le da el reloj verdadero; en las pruebas, uno **falso** que devuelve la hora que haga falta.

El reloj es una **clase abstracta** (`Reloj`): la clase real devuelve la hora del sistema y, en la prueba, `RelojFalso` devuelve la que la prueba decida.

**`tienda.dart`**

```dart
// Código que depende de la HORA. Si llamara a DateTime.now() directamente, sus pruebas dependerían de cuándo se ejecuten.
// Solución: recibir un Reloj desde fuera (inyección de dependencias) y, en las pruebas, darle uno falso.
abstract class Reloj {
  int get hora;
}

class RelojReal implements Reloj {
  @override
  int get hora => DateTime.now().hour;
}

class Tienda {
  final Reloj reloj;
  Tienda(this.reloj);

  String get saludo {
    final h = reloj.hora;
    if (h < 12) return 'buenos días';
    if (h < 20) return 'buenas tardes';
    return 'buenas noches';
  }

  bool get abierta => reloj.hora >= 9 && reloj.hora < 21;
}
```

**`tienda_test.dart`**

```dart
import 'package:carrito_app/tienda.dart';
import 'package:test/test.dart';

// Un reloj falso: la prueba decide qué hora es
class RelojFalso implements Reloj {
  @override
  final int hora;
  RelojFalso(this.hora);
}

void main() {
  test('por la mañana saluda con «buenos días» y la tienda está cerrada a las 8', () {
    final tienda = Tienda(RelojFalso(8));
    expect(tienda.saludo, 'buenos días');
    expect(tienda.abierta, isFalse);
  });

  test('por la tarde saluda con «buenas tardes» y está abierta', () {
    final tienda = Tienda(RelojFalso(15));
    expect(tienda.saludo, 'buenas tardes');
    expect(tienda.abierta, isTrue);
  });

  test('por la noche saluda con «buenas noches» y está cerrada', () {
    final tienda = Tienda(RelojFalso(22));
    expect(tienda.saludo, 'buenas noches');
    expect(tienda.abierta, isFalse);
  });

  test('abre a las 9 y cierra a las 21 (casos límite)', () {
    expect(Tienda(RelojFalso(8)).abierta, isFalse);
    expect(Tienda(RelojFalso(9)).abierta, isTrue);
    expect(Tienda(RelojFalso(20)).abierta, isTrue);
    expect(Tienda(RelojFalso(21)).abierta, isFalse);
  });
}
```

Al ejecutar las pruebas:

```text
+0: por la mañana saluda con «buenos días» y la tienda está cerrada a las 8
+1: por la tarde saluda con «buenas tardes» y está abierta
+2: por la noche saluda con «buenas noches» y está cerrada
+3: abre a las 9 y cierra a las 21 (casos límite)
+4: All tests passed!
```

Las pruebas dan el mismo resultado **a las 3 de la mañana y a las 3 de la tarde**, porque la hora la decide cada prueba. Sin el reloj falso habría que esperar al día siguiente para probar «buenas noches».

## Dobles de prueba

Un objeto que sustituye a otro en las pruebas se llama **doble de prueba**. Hay varios tipos:

| Tipo | Qué hace | Ejemplo |
|---|---|---|
| **Falso** (*fake*) | Una versión simple pero que funciona | Una base de datos en memoria; el reloj de arriba |
| **Sustituto** (*stub*) | Devuelve respuestas fijas | Un servicio que siempre contesta «aprobado» |
| **Espía / simulado** (*mock*) | Además, **registra cómo lo llamaron** | Comprobar que se envió exactamente un correo |

Empieza siempre por el más simple (un falso hecho a mano). Las bibliotecas de simulación solo hacen falta cuando hay muchas dependencias.

## Qué probar y qué no

| Prueba | Qué comprueba | Cuántas |
|---|---|---|
| **Unitarias** | Una función o clase **sola** (las de esta unidad) | Muchas: son rápidas y baratas |
| **De integración** | Varias piezas juntas (por ejemplo, tu código con una base de datos real) | Menos |
| **De extremo a extremo** | El programa entero, como lo usaría una persona | Pocas: son lentas y frágiles |

!!! tip "Escribir la prueba primero"
    En el **desarrollo guiado por pruebas** (*TDD*) el ciclo es: 1) escribes una prueba que **falla**, 2) escribes el código **mínimo** para que pase, 3) **mejoras** el código sin que las pruebas dejen de pasar. Obliga a pensar primero qué debe hacer el código y deja una red de seguridad desde el principio.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Código que llama a la hora, al azar o a la red directamente | Recíbelo como parámetro o dependencia |
| Pruebas que fallan «a veces» | Busca lo que no controlas: hora, azar, orden, red |
| Falsos tan complicados que hay que probarlos a ellos | Mantenlos mínimos |
| Probar detalles internos en lugar del comportamiento | Prueba lo que el código **hace**, no cómo lo hace |

## Para practicar

El ejercicio [A4.6](ejercicios.md) pide probar un saludo con un reloj falso. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
