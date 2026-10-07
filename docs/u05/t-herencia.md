# 5.A Herencia

La **herencia** permite crear una clase nueva **a partir de otra ya existente**, aprovechando todo lo que esta tiene y añadiendo o cambiando lo que haga falta. Se usa cuando entre dos clases hay una relación **«es un»**: un perro **es un** animal; un gato **es un** animal.

| Término | Significa | Ejemplo |
|---|---|---|
| **Superclase** (o clase base, o padre) | La clase de la que se hereda | `Animal` |
| **Subclase** (o clase derivada, o hija) | La clase que hereda y la especializa | `Perro`, `Gato` |
| **Sobrescribir** (*override*) | Redefinir un método heredado para que haga otra cosa | `hablar()` |

## Un ejemplo

```dart
class Animal {
  final String nombre;
  Animal(this.nombre);

  String hablar() => '...';

  String presentarse() => '$nombre dice: ${hablar()}';
}

class Perro extends Animal {
  Perro(super.nombre);

  @override
  String hablar() => 'Guau';

  String traerPelota() => '$nombre trae la pelota';
}

class Gato extends Animal {
  Gato(super.nombre);

  @override
  String hablar() => 'Miau';

  @override
  String presentarse() => '${super.presentarse()} (y se hace el distraído)';
}

void main() {
  final animales = [Perro('Rex'), Gato('Misi'), Animal('Bicho')];
  for (final a in animales) {
    print(a.presentarse());
  }
  for (final a in animales) {
    if (a is Perro) {
      print(a.traerPelota());
    }
  }
}
```

Salida:

```text
Rex dice: Guau
Misi dice: Miau (y se hace el distraído)
Bicho dice: ...
Rex trae la pelota
```

Qué ocurre aquí:

* `Perro` y `Gato` **no repiten** el atributo `nombre` ni el método `presentarse`: los reciben de `Animal`.
* Cada subclase **sobrescribe `hablar()`** para dar su propio sonido; `Animal` tiene una versión genérica (`...`), y `Bicho`, que es un `Animal` normal, la usa.
* `Gato` además sobrescribe `presentarse()` y **reutiliza** la versión del padre con `super`, añadiéndole algo.
* `Perro` añade un método que `Animal` no tiene (`traerPelota`). Para llamarlo hay que **comprobar el tipo** antes, porque la lista contiene animales de todo tipo.
* Las tres líneas se imprimen con **la misma llamada** (`a.presentarse()`), pero el resultado depende de **qué objeto concreto** hay detrás. Eso se llama **polimorfismo** y se explica en el [siguiente apartado](t-abstractas.md).

## Cómo se escribe en Dart

| Necesito... | En Dart |
|---|---|
| Heredar | `class Perro extends Animal { ... }` |
| Constructor del padre | `Perro(super.nombre);` (o `Perro(String n) : super(n);`). Los constructores **no se heredan**: cada clase define los suyos |
| Sobrescribir un método | Repetirlo con el mismo nombre y `@override`; el analizador avisa si falta |
| Llamar a la versión del padre | `super.presentarse()` |
| Comprobar el tipo | `a is Perro` (y dentro del `if` Dart ya trata `a` como `Perro`) |
| Impedir que hereden de ti | `final class Perro` |
| ¿Cuántos padres? | **Uno** (herencia simple); se completa con interfaces y *mixins* |

## ¿Herencia o composición?

La herencia es muy cómoda, pero crea un vínculo fuerte: si la clase base cambia, todas las hijas se ven afectadas. Antes de heredar, pregúntate si la relación es de verdad «es un».

| Relación | Se resuelve con | Ejemplo |
|---|---|---|
| **«es un»** (un perro es un animal) | Herencia | `Perro extends Animal` |
| **«tiene un»** (un coche tiene un motor) | **Composición**: un atributo con otro objeto (ver [4.D](../u04/t-colecciones.md)) | `Coche` con un atributo `Motor` |
| **«sabe hacer»** (un pato sabe volar) | **Interfaz** (ver [5.C](t-interfaces.md)) | `Pato` implementa `Volador` |

!!! warning "No heredes solo para reutilizar código"
    Si `Pila` heredara de `Lista` solo para aprovechar `add`, una pila **no es** una lista: acabaría ofreciendo operaciones que no tienen sentido en ella. En ese caso, la pila **tiene** una lista dentro (composición). Regla práctica: ante la duda, **composición**.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Heredar cuando la relación no es «es un» | Probar la frase «un A es un B» en voz alta |
| Jerarquías muy profundas (A → B → C → D → E) | Mantenerlas cortas; más de tres niveles suele ser mal signo |
| Llamar a un método de la subclase desde una variable del tipo base | Comprobar el tipo antes, o replantear el diseño |
| Olvidar inicializar la parte heredada en el constructor | Llamar siempre al constructor del padre con lo que necesite |

Olvidar `@override` no impide que compile, pero el analizador lo marca: acostúmbrate a escribirlo siempre.

## Para practicar

Los ejercicios de herencia y sobrescritura están en [U5.1](herencia.md) (por ejemplo, los de artículos y vehículos). Compara cómo se escribe en otro lenguaje con [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
