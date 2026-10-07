# 6.A Diseñar una jerarquía de clases

En la [unidad 5](../u05/index.md) viste **cómo se escribe** la herencia, las clases abstractas y las interfaces. Aquí se trata de **decidir bien**: qué clases hacen falta, qué va en la base y qué en cada hija, y cómo se organizan los constructores para que el diseño aguante cambios.

## Un ejemplo completo: un hotel

```dart
abstract class Habitacion {
  final int numero;
  bool libre = true;

  Habitacion(this.numero);

  String tipo();
  int precio();

  String describir() =>
      '$numero ${tipo()}: ${precio()} €/noche${libre ? '' : ' (ocupada)'}';
}

class Individual extends Habitacion {
  Individual(super.numero);

  @override
  String tipo() => 'Individual';

  @override
  int precio() => 50;
}

class Doble extends Habitacion {
  final bool desayuno;
  Doble(super.numero, {this.desayuno = false});

  @override
  String tipo() => desayuno ? 'Doble con desayuno' : 'Doble';

  @override
  int precio() => desayuno ? 90 : 80;
}

class Suite extends Habitacion {
  final int extras;
  Suite(super.numero, {this.extras = 0});

  @override
  String tipo() => 'Suite';

  @override
  int precio() => 150 + 20 * extras;
}

class Hotel {
  final List<Habitacion> _habitaciones = [];

  List<Habitacion> get todas => List.unmodifiable(_habitaciones);

  void agregar(Habitacion habitacion) => _habitaciones.add(habitacion);

  void reservar(int numero) {
    final habitacion = _habitaciones.where((h) => h.numero == numero).firstOrNull;
    if (habitacion == null) {
      throw ArgumentError('la habitación $numero no existe');
    }
    if (!habitacion.libre) {
      throw StateError('la habitación $numero ya está ocupada');
    }
    habitacion.libre = false;
  }

  List<Habitacion> libres() => _habitaciones.where((h) => h.libre).toList();

  int precioMedio() {
    var suma = 0;
    for (final h in _habitaciones) {
      suma += h.precio();
    }
    return suma ~/ _habitaciones.length;
  }
}

void main() {
  final hotel = Hotel();
  hotel.agregar(Individual(101));
  hotel.agregar(Doble(201, desayuno: true));
  hotel.agregar(Suite(301, extras: 2));
  for (final h in hotel.todas) {
    print(h.describir());
  }

  hotel.reservar(201);
  print('tras reservar la 201: ${hotel.todas[1].describir()}');
  try {
    hotel.reservar(201);
  } on StateError catch (e) {
    print('Error: ${e.message}');
  }
  try {
    hotel.reservar(999);
  } on ArgumentError catch (e) {
    print('Error: ${e.message}');
  }
  print('libres: ${hotel.libres().map((h) => h.numero).join(', ')}');
  print('precio medio: ${hotel.precioMedio()} €');
}
```

Salida:

```text
101 Individual: 50 €/noche
201 Doble con desayuno: 90 €/noche
301 Suite: 190 €/noche
tras reservar la 201: 201 Doble con desayuno: 90 €/noche (ocupada)
Error: la habitación 201 ya está ocupada
Error: la habitación 999 no existe
libres: 101, 301
precio medio: 110 €
```

Cómo se ha decidido el diseño:

* **La base lleva lo que es común a todas.** `Habitacion` tiene el `numero` y si está `libre`, que valen para cualquier habitación, y un método `describir()` ya escrito.
* **Cada hija define solo lo que cambia.** El `tipo()` y el `precio()` son **abstractos**: cada habitación sabe el suyo. `Doble` y `Suite` añaden sus propios datos (`desayuno`, `extras`).
* **`describir()` usa los métodos abstractos.** Es el patrón del [método plantilla](../u05/t-abstractas.md): la base fija el esquema, las hijas rellenan los huecos.
* **El hotel solo conoce `Habitacion`.** Si se añade un tipo nuevo, `Hotel` no cambia: es polimorfismo y composición («un hotel **tiene** habitaciones»).
* **Las reglas, dentro de su sitio.** Reservar comprueba que la habitación exista y esté libre, y lanza una excepción **antes** de modificar nada. Además, `todas` devuelve `List.unmodifiable(...)`, una vista que **no deja modificar** la lista interna.
* **Los constructores reparten el trabajo.** La base inicializa el número; cada hija pasa ese dato hacia arriba y se queda con el suyo.

## Modificadores que ayudan a diseñar

| Necesito... | En Dart |
|---|---|
| Clase que no se puede instanciar | `abstract class` |
| Clase de la que no se puede heredar | `final class` |
| Solo contrato (se implementa, no se hereda) | `abstract interface class` |
| Jerarquía cerrada | `sealed class` |
| Visible solo en el archivo | nombre con guion bajo `_numero` |
| Valor que no cambia tras crear el objeto | `final` |

## Preguntas antes de crear una jerarquía

| Pregunta | Si la respuesta es... |
|---|---|
| ¿La relación es «es un»? | No → composición o interfaz, no herencia |
| ¿Hay código o datos comunes? | Sí → clase base (abstracta si no tiene sentido por sí sola) |
| ¿Solo hay que fijar un contrato? | Sí → interfaz |
| ¿Lo que cambia es un cálculo o una regla? | Valorar pasar una **función** (ver [4.E](../u04/t-modelar.md)) |
| ¿Las variantes son un conjunto cerrado con datos distintos? | Jerarquía cerrada (ver [5.D](../u05/t-sealed.md)) |
| ¿Se podrá usar cualquier hija donde se pida la base sin sorpresas? | Si no → hay que rediseñar (ver [6.C](t-solid-2.md), principio L) |

## Buenas prácticas

* **Jerarquías cortas y anchas** (una base y varias hijas) mejor que largas y estrechas.
* **La base, estable.** Cuanto más se cambia, más hijas se rompen. Si una base cambia a menudo, probablemente tiene demasiadas responsabilidades.
* **Cierra lo que no necesites abrir** (`final`, `sealed`, el comportamiento por defecto de Kotlin): permitir heredar es una decisión, no un valor por defecto.
* **No expongas tus colecciones internas**: ofrece métodos o una vista de solo lectura.
* **Prefiere la composición** cuando dudes entre heredar o tener un atributo.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Una clase base con atributos que a algunas hijas no les sirven | Mover esos atributos a las hijas que los usan |
| Hijas que sobrescriben un método solo para «no hacer nada» o lanzar un error | Señal de que la jerarquía está mal (ver 6.C) |
| Repetir el mismo código en varias hijas | Subirlo a la base |
| Una base con métodos que solo usa una hija | Bajarlos a esa hija |

## Para practicar

Haz los ejercicios de [U6.1 · Jerarquía de clases](jerarquia.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
