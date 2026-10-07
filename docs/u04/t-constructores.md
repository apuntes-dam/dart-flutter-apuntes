# 4.C Constructores y enumerados

## Más de una forma de crear un objeto

A veces un objeto se puede crear de varias maneras: con unos valores por defecto, con datos propios o a partir de una «receta» ya preparada. Cada lenguaje lo resuelve a su manera.

Dart ofrece varias herramientas: **parámetros posicionales opcionales** entre corchetes con valor por defecto (`[this.nombre = 'Margarita']`), **parámetros con nombre** (`{{String nombre = 'Margarita'}}`) y **constructores con nombre** (`Pizza.cuatroQuesos(...)`), que sustituyen a la sobrecarga: Dart **no permite** dos constructores o dos métodos con el mismo nombre. Lo que no llega como parámetro se asigna en la **lista de inicialización** (`: extras = 0`), antes de ejecutar el cuerpo.

```dart
enum Tamano {
  pequena('Pequeña', 6),
  mediana('Mediana', 8),
  grande('Grande', 10);

  final String etiqueta;
  final int precioBase;
  const Tamano(this.etiqueta, this.precioBase);
}

class Pizza {
  final Tamano tamano;
  final String nombre;
  int extras;

  Pizza(this.tamano, [this.nombre = 'Margarita']) : extras = 0;

  Pizza.cuatroQuesos(this.tamano)
      : nombre = 'Cuatro quesos',
        extras = 3;

  void anadirExtra([int cantidad = 1]) {
    extras += cantidad;
  }

  int precio() => tamano.precioBase + extras;

  @override
  String toString() => 'Pizza($nombre, ${tamano.etiqueta}, $extras extras, ${precio()} €)';
}

void main() {
  final a = Pizza(Tamano.mediana);
  final b = Pizza.cuatroQuesos(Tamano.grande);
  final c = Pizza(Tamano.pequena, 'Barbacoa');
  print(a);
  print(b);
  print(c);
  a.anadirExtra();
  a.anadirExtra(2);
  print(a);
  print('total del pedido: ${a.precio() + b.precio() + c.precio()} €');
  print('tamaños: ${Tamano.values.map((t) => t.etiqueta).join(', ')}');
}
```

Salida:

```text
Pizza(Margarita, Mediana, 0 extras, 8 €)
Pizza(Cuatro quesos, Grande, 3 extras, 13 €)
Pizza(Barbacoa, Pequeña, 0 extras, 6 €)
Pizza(Margarita, Mediana, 3 extras, 11 €)
total del pedido: 30 €
tamaños: Pequeña, Mediana, Grande
```

En este ejemplo hay tres formas de obtener una pizza: **solo con el tamaño** (el nombre por defecto es «Margarita»), **con nombre propio** (`Barbacoa`) y la receta ya preparada con un **constructor alternativo** (`cuatroQuesos`, que además trae tres extras). Después, `anadirExtra` se llama **con y sin argumento**.

Para un método que se puede llamar con o sin argumento, como `anadirExtra()` y `anadirExtra(2)`, se usa **un solo método** con un parámetro opcional con valor por defecto (`[int cantidad = 1]`).

!!! tip "Un solo sitio para las reglas"
    Procura que **todos los constructores acaben pasando por el mismo código** (un constructor principal al que los demás llaman): así cualquier regla que añadas más adelante (por ejemplo, un máximo de extras) se escribe **una sola vez**.

## Enumerados

Un **enumerado** es un tipo con un **conjunto fijo y cerrado de valores**: los días de la semana, los estados de un pedido, los tamaños de una pizza. Es mejor que usar números o textos sueltos porque el compilador impide valores inventados (`Tamano.gigante` no existe) y el código se lee solo.

Un `enum` en Dart puede tener **campos, constructor y métodos**, con constructor `const`. `Tamano.values` devuelve todos los valores en orden.

En el ejemplo, cada tamaño lleva **datos asociados** (la etiqueta que se muestra y el precio base), de modo que el resto del programa no necesita un `if` por cada tamaño: pregunta el dato al propio valor.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Repetir las comprobaciones en cada constructor | Un constructor principal y los demás que lo llaman |
| Dos constructores casi iguales con distinto orden de parámetros | Usa valores por defecto o un método de fábrica con nombre claro |
| Usar números o textos para representar categorías (`tipo = 1`) | Un enumerado |
| Cadenas de `if`/`else` según el valor de un enumerado | Pon el dato o el comportamiento **dentro** del enumerado |

## Para practicar

Los constructores, los enumerados y la sobrecarga se practican en [U4.6 · Prueba](prueba.md). Compara cómo se escribe en otro lenguaje con [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
