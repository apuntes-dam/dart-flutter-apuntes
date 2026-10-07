# 5.B Polimorfismo y clases abstractas

## Polimorfismo

**Polimorfismo** significa «muchas formas»: una misma llamada se comporta de forma distinta según el **objeto real** sobre el que se hace. En el ejemplo de la [herencia](t-herencia.md), `a.presentarse()` imprimía algo diferente para un perro, un gato y un animal genérico.

Para entenderlo hay que distinguir dos tipos:

* El **tipo declarado** de la variable, que es lo que ve el compilador (`Animal`).
* El **tipo real** del objeto, que se decide al crearlo (`Perro`, `Gato`...).

Al llamar a un método **sobrescrito**, se ejecuta la versión del **tipo real**. Gracias a eso se puede escribir código que trabaja con el tipo general y funciona con todos los casos presentes y **futuros**: si mañana se añade `Loro`, el bucle que recorre los animales no cambia.

## Clases abstractas

A veces una clase base solo tiene sentido **como molde**: un «instrumento» genérico no suena a nada, pero sí una guitarra o un tambor. Una **clase abstracta** es una clase que **no se puede instanciar** y que obliga a sus subclases a completar lo que falta.

Se marca la clase con **`abstract`**. Un método **sin cuerpo** (`String sonido();`) es abstracto y las subclases **deben** implementarlo. No se puede crear un objeto de una clase abstracta (`Instrumento('x')` es un error de compilación).

```dart
abstract class Instrumento {
  final String nombre;
  Instrumento(this.nombre);

  String sonido(); // abstracto: cada subclase lo define

  String tocar() => '$nombre: ${sonido()}'; // concreto: usa el abstracto
}

class Guitarra extends Instrumento {
  Guitarra() : super('Guitarra');

  @override
  String sonido() => 'rasgueo de cuerdas';
}

class Tambor extends Instrumento {
  Tambor() : super('Tambor');

  @override
  String sonido() => '¡pum pum!';
}

class Piano extends Instrumento {
  Piano() : super('Piano');

  @override
  String sonido() => 'notas de teclado';
}

void main() {
  final orquesta = <Instrumento>[Guitarra(), Tambor(), Piano()];
  for (final instrumento in orquesta) {
    print(instrumento.tocar());
  }
  print('en la orquesta hay ${orquesta.length} instrumentos');
}
```

Salida:

```text
Guitarra: rasgueo de cuerdas
Tambor: ¡pum pum!
Piano: notas de teclado
en la orquesta hay 3 instrumentos
```

Fíjate en el diseño:

* `sonido()` es **abstracto**: la clase base sabe que todo instrumento suena, pero no *cómo*, así que lo deja sin escribir.
* `tocar()` es **concreto** y ya está escrito, pero **usa** `sonido()`. Es una idea muy útil: la base fija **el esquema** y cada subclase rellena el hueco. Se la conoce como método plantilla.
* El código que recorre la orquesta solo conoce `Instrumento`; no le importa cuántos tipos hay.

!!! tip "¿Cuándo hacer una clase abstracta?"
    Cuando la clase base **comparte código o datos** (aquí, el nombre y `tocar`) y además **obliga** a las hijas a definir algo. Si solo hay que fijar un contrato, sin código común, una [interfaz](t-interfaces.md) es más flexible.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Intentar crear un objeto de una clase abstracta | Crear un objeto de una subclase concreta |
| Que una subclase olvide implementar un método abstracto | El compilador (o Python al crear el objeto) lo avisa: hay que implementarlo todo |
| Poner demasiado en la clase abstracta | Solo lo que es **común** a todas las hijas |
| Usar herencia solo para compartir código | Valorar composición o funciones auxiliares |

## Para practicar

Haz los ejercicios de clases abstractas y polimorfismo de [U5.1](herencia.md) (los primeros). Y para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
