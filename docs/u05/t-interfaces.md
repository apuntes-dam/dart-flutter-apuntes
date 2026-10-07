# 5.C Interfaces

Una **interfaz** es un **contrato**: una lista de métodos que una clase **promete ofrecer**, sin decir cómo. Cualquier clase que cumpla el contrato puede usarse allí donde se pida la interfaz, sin importar de qué clase venga.

La herencia responde a «**qué es** el objeto» (un pato **es un** animal). La interfaz responde a «**qué sabe hacer**» (un pato **sabe volar** y **sabe nadar**). Y eso es independiente: un avión también vuela, aunque no tenga nada que ver con un animal.

En Dart toda clase es también una interfaz implícita, pero desde Dart 3 se declara **explícitamente** con **`abstract interface class`**, que solo se puede **implementar** (`implements`), no heredar. Lo que se quiere **compartir con código ya hecho** se escribe en un **`mixin`**, que se añade a una clase con **`with`**. En el ejemplo, `Nadador` es un contrato puro (`implements`) y `Volador` es un `mixin`: aporta `aterrizar()` ya escrito y obliga a escribir `volar()`. Una clase puede usar **varios** `mixin` e implementar **varias** interfaces.

```dart
abstract interface class Nadador {
  String nadar();
}

mixin Volador {
  String volar();

  String aterrizar() => 'aterriza suavemente';
}

class Pato with Volador implements Nadador {
  @override
  String volar() => 'El pato bate las alas';

  @override
  String nadar() => 'El pato chapotea';
}

class Avion with Volador {
  @override
  String volar() => 'El avión despega';

  @override
  String aterrizar() => 'aterriza con tren de aterrizaje';
}

class Pez implements Nadador {
  @override
  String nadar() => 'El pez ondula';
}

void main() {
  final voladores = <Volador>[Pato(), Avion()];
  for (final v in voladores) {
    print('${v.volar()} y ${v.aterrizar()}');
  }
  final nadadores = <Nadador>[Pato(), Pez()];
  for (final n in nadadores) {
    print(n.nadar());
  }
}
```

Salida:

```text
El pato bate las alas y aterriza suavemente
El avión despega y aterriza con tren de aterrizaje
El pato chapotea
El pez ondula
```

Qué enseña este ejemplo:

* **`Pato` cumple dos contratos** (`Volador` y `Nadador`). Una clase solo puede tener **un** padre, pero puede cumplir **muchas** interfaces.
* **`Avion` y `Pato` no se parecen en nada**, pero ambos son `Volador`: una lista de voladores los mezcla sin problema. Es polimorfismo, ahora **sin herencia**.
* `Pato` usa tal cual el `aterrizar()` por defecto; `Avion` lo **sobrescribe** con el suyo.
* Quien recibe un `Volador` solo puede llamar a lo que **el contrato** promete, aunque el objeto real tenga más cosas.

## Clase abstracta o interfaz

| | Clase abstracta | Interfaz |
|---|---|---|
| Relación | «es un» (hereda) | «sabe hacer» (cumple un contrato) |
| ¿Cuántas puede tener una clase? | **Una** | **Todas las que quiera** |
| ¿Guarda datos (atributos)? | Sí | No (o muy limitado) |
| ¿Puede traer código? | Sí | Solo comportamiento por defecto |
| Cuándo usarla | Familia de clases con **código y datos comunes** | Capacidad que pueden tener **clases sin relación** |

!!! tip "Programa contra el contrato"
    Escribe funciones que reciban la interfaz (`Volador`) y no la clase concreta (`Pato`): serán reutilizables con cualquier clase futura que cumpla el contrato. Es una de las ideas más importantes del diseño con objetos, y la base de los [principios SOLID](../u06/index.md).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Interfaces enormes con métodos que muchas clases no necesitan | Interfaces pequeñas, con **una** capacidad cada una |
| Usar una interfaz cuando hay mucho código común | Clase abstracta |
| No implementar todos los métodos | El compilador lo avisa (en Python, al crear el objeto) |
| Poner estado dentro de la interfaz | Un contrato describe **qué hacer**, no qué guardar |

## Para practicar

Haz los ejercicios de interfaces de [U5.1](herencia.md) (dispositivos y notificaciones). Compara cómo se escribe en otro lenguaje con [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
