# A5.A Patrones de creación

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 6 (clases, herencia, interfaces y SOLID) y [A1](../a1/index.md) (funciones como valores).

Un **patrón de diseño** es una solución conocida a un problema que se repite al diseñar programas. No es una librería ni código que copiar: es una **forma de organizar clases** que otros programadores ya reconocen. Saber su nombre sirve para algo muy práctico: decir «esto es una fábrica» comunica en tres palabras lo que sin él costaría un párrafo.

Los patrones **de creación** tratan de **cómo se crean los objetos**. Todos los ejemplos de esta unidad usan una cafetería.

## Singleton: una sola instancia

Es una clase de la que **solo puede existir un objeto**, y todos reciben el mismo. Sirve para algo realmente único: la configuración de la aplicación, un registro de mensajes.

En Dart se hace con un **constructor privado** (`Configuracion._`) y un constructor **`factory`**, que a diferencia de uno normal puede devolver un objeto que ya existe. La instancia única es una variable `static final`, que se crea la primera vez que se usa. Con `identical(a, b)` se comprueba que son el mismo objeto.

```dart
// Singleton: una clase de la que solo puede existir UNA instancia (aquí, la configuración de la cafetería).
class Configuracion {
  final String local;
  final int iva;

  Configuracion._(this.local, this.iva); // constructor privado: nadie de fuera puede crear otra

  static final Configuracion _unica = Configuracion._('Café Central', 10);

  factory Configuracion() => _unica; // un constructor «factory» puede devolver un objeto que ya existe
}

void main() {
  final a = Configuracion();
  final b = Configuracion();
  print('misma instancia: ${identical(a, b) ? "sí" : "no"}');
  print('local: ${a.local}, IVA ${a.iva}%');
}
```

Salida:

```text
misma instancia: sí
local: Café Central, IVA 10%
```

!!! warning "El patrón más discutido"
    Un singleton es **estado global con buena presentación**: cualquier parte del programa puede cambiarlo, es difícil de sustituir en las pruebas ([A4](../a4/t-aislar.md)) y esconde dependencias. Úsalo solo si de verdad **tiene que** haber uno, y si puedes, **pásalo como parámetro** en lugar de pedírselo a la clase desde cualquier sitio.

## Fábrica: decidir qué clase crear

Una **fábrica** es una función que, a partir de un dato (aquí un nombre), **decide qué clase concreta crear**. Quien la usa solo conoce el tipo común (`Bebida`), no las clases concretas: añadir un tipo nuevo no obliga a cambiar el resto del programa.

El `switch` con expresiones devuelve la clase que toca y obliga a cubrir el caso por defecto (`_`).

```dart
// Fábrica: una función decide QUÉ clase concreta crear, y quien la usa solo conoce el tipo común (Bebida).
abstract class Bebida {
  String get nombre;
  int get precio; // en céntimos
}

class Cafe implements Bebida {
  @override
  String get nombre => 'café';
  @override
  int get precio => 150;
}

class Te implements Bebida {
  @override
  String get nombre => 'té';
  @override
  int get precio => 120;
}

class Zumo implements Bebida {
  @override
  String get nombre => 'zumo';
  @override
  int get precio => 200;
}

Bebida fabricar(String nombre) => switch (nombre) {
      'café' => Cafe(),
      'té' => Te(),
      'zumo' => Zumo(),
      _ => throw ArgumentError('$nombre: no está en la carta'),
    };

void main() {
  for (final nombre in ['café', 'té', 'zumo', 'chocolate']) {
    try {
      final bebida = fabricar(nombre);
      print('${bebida.nombre}: ${bebida.precio} céntimos');
    } on ArgumentError catch (e) {
      print(e.message);
    }
  }
}
```

Salida:

```text
café: 150 céntimos
té: 120 céntimos
zumo: 200 céntimos
chocolate: no está en la carta
```

Es la «D» de SOLID ([unidad 6](../../u06/t-solid-2.md)) en acción: el código depende de una **abstracción** (`Bebida`), no de las clases concretas. Y los datos desconocidos se rechazan **en un solo lugar**, con un mensaje claro.

## Builder: construir paso a paso

Un **builder** construye un objeto con **muchos datos opcionales**, con llamadas encadenadas que dicen qué es cada cosa. Lo que no se indica toma su valor por defecto.

En Dart los **parámetros con nombre** (`Pedido(bebida: 'capuchino', leche: 'avena')`) resuelven casi siempre el mismo problema sin escribir un builder. Úsalo cuando construir tenga **pasos o reglas** (validar al final, calcular valores derivados).

```dart
// Builder: construir un objeto con muchos datos opcionales paso a paso, con llamadas encadenadas.
// (En Dart también valdrían los parámetros con nombre; el builder se usa cuando la construcción tiene reglas o pasos.)
class Pedido {
  final String bebida;
  final String tamano;
  final String leche;
  final bool azucar;

  Pedido._(this.bebida, this.tamano, this.leche, this.azucar);

  @override
  String toString() => '$bebida ($tamano), leche: $leche, ${azucar ? "con" : "sin"} azúcar';
}

class PedidoBuilder {
  String _bebida = 'café';
  String _tamano = 'pequeño';
  String _leche = 'ninguna';
  bool _azucar = false;

  PedidoBuilder bebida(String b) {
    _bebida = b;
    return this; // devolver «this» permite encadenar
  }

  PedidoBuilder tamano(String t) {
    _tamano = t;
    return this;
  }

  PedidoBuilder leche(String l) {
    _leche = l;
    return this;
  }

  PedidoBuilder conAzucar() {
    _azucar = true;
    return this;
  }

  Pedido construir() => Pedido._(_bebida, _tamano, _leche, _azucar);
}

void main() {
  print(PedidoBuilder().bebida('capuchino').tamano('grande').leche('avena').construir());
  print(PedidoBuilder().conAzucar().construir()); // lo que no se indica toma el valor por defecto
}
```

Salida:

```text
capuchino (grande), leche: avena, sin azúcar
café (pequeño), leche: ninguna, con azúcar
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar un singleton «porque es cómodo» | Pregúntate si de verdad tiene que haber una sola instancia; si no, pásala como parámetro |
| Una fábrica con un `switch` enorme que crece sin parar | Cuando crezca, usa un diccionario o un registro de clases |
| Un builder para una clase de dos campos | Si bastan parámetros con nombre o un constructor, no lo uses |
| Olvidar validar al final de la construcción | La validación va en `construir()`: un objeto a medias no debería existir |

## Para practicar

Los ejercicios [A5.1, A5.2 y A5.3](ejercicios.md) piden una fábrica de figuras, un registro único y un correo con builder. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
