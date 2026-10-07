# A1.C Componer, esperar e inmutabilidad

Tres ideas que completan la programación funcional: **componer** funciones pequeñas en otras mayores, **calcular solo lo necesario** (evaluación perezosa) y **no modificar los datos** (inmutabilidad).

## Componer funciones

Si tienes funciones pequeñas que hacen una cosa cada una, puedes unirlas: la salida de una es la entrada de la siguiente. Eso es una **tubería**.

En Dart se compone a mano con una función que recibe `f` y `g` y devuelve `(x) => f(g(x))`; los **genéricos** `<A, B, T>` dicen qué tipo entra y sale en cada una. Para encadenar muchos pasos, `fold` sobre la lista de funciones aplica cada una al resultado de la anterior. La **aplicación parcial** fija un argumento y devuelve la función que espera el resto.

```dart
// Funciones de orden superior: componer funciones, encadenarlas, fijar argumentos y devolver funciones.
int doble(int n) => n * 2;
int sumar3(int n) => n + 3;

// componer(f, g)(x) = f(g(x)); los genéricos <A, B, T> dicen qué tipos entran y salen
T Function(A) componer<A, B, T>(T Function(B) f, B Function(A) g) => (x) => f(g(x));

// una "tubería": aplica las funciones de la lista una tras otra
String Function(String) tuberia(List<String Function(String)> pasos) =>
    (texto) => pasos.fold(texto, (acc, paso) => paso(acc));

// aplicación parcial: fija el primer argumento de una función de dos
int Function(int) parcial(int Function(int, int) f, int a) => (b) => f(a, b);

// función que devuelve otra función (currying): potencia(base)(exponente)
int Function(int) potencia(int base) => (exp) {
      var resultado = 1;
      for (var i = 0; i < exp; i++) {
        resultado *= base;
      }
      return resultado;
    };

void main() {
  print('componer(doble, sumar3)(4) = ${componer(doble, sumar3)(4)}');

  final slug = tuberia([
    (s) => s.trim(),
    (s) => s.toLowerCase(),
    (s) => s.replaceAll(RegExp(r'\s+'), '-'),
  ]);
  print('slug: "  Hola Mundo Cruel  " -> "${slug("  Hola Mundo Cruel  ")}"');

  final sumar5 = parcial((a, b) => a + b, 5);
  print('sumar5(10) = ${sumar5(10)}');
  print('potencia(2)(10) = ${potencia(2)(10)}');
}
```

Salida:

```text
componer(doble, sumar3)(4) = 14
slug: "  Hola Mundo Cruel  " -> "hola-mundo-cruel"
sumar5(10) = 15
potencia(2)(10) = 1024
```

La función `slug` convierte un título en una dirección web: quita los espacios de los extremos, pasa a minúsculas y cambia los espacios por guiones. Cada paso es una función de una línea, fácil de probar por separado.

## Calcular solo lo necesario

Una colección **ansiosa** calcula todos sus elementos al crearse. Una **perezosa** calcula cada elemento cuando alguien lo pide, y por eso puede ser **infinita**. En Dart:

En Dart, `where` y `map` ya son perezosos. Un generador **`sync*`** con `yield` produce valores uno a uno: la función se queda pausada en cada `yield` hasta que se pide el siguiente. Así se puede describir una secuencia **infinita** (`naturales`) y tomar solo los que hacen falta con `take`.

```dart
// Evaluación perezosa: los valores se calculan solo cuando alguien los pide.

// sync* define un generador: cada yield entrega un valor y la función se queda "en pausa" hasta el siguiente
Iterable<int> naturales() sync* {
  var n = 1;
  while (true) {
    yield n++; // una secuencia infinita: nunca termina por sí sola
  }
}

Iterable<int> fibonacci() sync* {
  var a = 0, b = 1;
  while (true) {
    yield a;
    final siguiente = a + b;
    a = b;
    b = siguiente;
  }
}

void main() {
  // where y map no calculan nada todavía: solo describen la tubería
  final pares = naturales().where((n) {
    print('revisando $n');
    return n.isEven;
  }).map((n) => n * n);
  print('(se define la secuencia: todavía no se ha calculado nada)');

  // take(3) pide valores uno a uno hasta tener tres; aquí se hace el trabajo
  print('primeros 3 pares al cuadrado: ${pares.take(3).toList()}');
  print('fibonacci: ${fibonacci().take(8).toList()}');
}
```

Salida:

```text
(se define la secuencia: todavía no se ha calculado nada)
revisando 1
revisando 2
revisando 3
revisando 4
revisando 5
revisando 6
primeros 3 pares al cuadrado: [4, 16, 36]
fibonacci: [0, 1, 1, 2, 3, 5, 8, 13]
```

Mira el orden de la salida. Primero aparece el aviso de que la secuencia está definida pero sin calcular; después, `revisando 1` a `revisando 6`: solo se revisaron los números necesarios para encontrar **tres pares**, y ni uno más. Eso es la evaluación perezosa.

## No modificar los datos

Una función es **pura** si su resultado depende solo de sus argumentos y no cambia nada fuera de ella. Las puras son más fáciles de entender y de probar, y también de usar **a la vez** en varios hilos (lo verás en A3). La forma de conseguirlo con datos es la **inmutabilidad**: en vez de cambiar un objeto, se crea otro con el cambio.

Un campo `final` no se puede reasignar; con todos los campos `final` y un constructor `const`, el objeto es inmutable. Para «cambiar» algo se escribe un método que devuelve **otro objeto** (`conPuntos`). `List.unmodifiable` crea una lista que lanza un error si intentas añadir o quitar.

```dart
// Inmutabilidad: en vez de modificar un objeto, se crea una copia con el cambio.
class Jugador {
  final String nombre;
  final int puntos;
  const Jugador(this.nombre, this.puntos);

  // devuelve un Jugador nuevo; el original no cambia
  Jugador conPuntos(int nuevos) => Jugador(nombre, nuevos);
}

String describir(Jugador j) => '${j.nombre} tiene ${j.puntos} puntos';

// función pura: el resultado depende solo de los argumentos y no toca nada de fuera
int sumaPura(int a, int b) => a + b;

// función impura: lee y modifica una variable externa; el mismo argumento da resultados distintos
int total = 0;
int sumarAlTotal(int x) {
  total += x;
  return total;
}

void main() {
  const original = Jugador('Ana', 10);
  final actualizado = original.conPuntos(15);
  print('original: ${describir(original)}');
  print('actualizado: ${describir(actualizado)}');
  print('original sigue igual: ${describir(original)}');

  final lista = List.unmodifiable([1, 2, 3]); // add() o remove() lanzarían un error
  final nueva = [...lista, 4];
  print('lista original: $lista');
  print('lista nueva: $nueva');

  print('pura: ${sumaPura(2, 3)} y ${sumaPura(2, 3)}');
  print('impura: ${sumarAlTotal(3)} y ${sumarAlTotal(3)}');
}
```

Salida:

```text
original: Ana tiene 10 puntos
actualizado: Ana tiene 15 puntos
original sigue igual: Ana tiene 10 puntos
lista original: [1, 2, 3]
lista nueva: [1, 2, 3, 4]
pura: 5 y 5
impura: 3 y 6
```

Fíjate en las dos últimas líneas: la función **pura** da `5` las dos veces; la **impura** da `3` y luego `6`, porque guarda estado fuera de la función y el mismo argumento produce resultados distintos.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Esperar que una secuencia perezosa «ya esté calculada» | Recuerda que solo se calcula al pedirla; si hay efectos secundarios (como imprimir), salen más tarde |
| Recorrer una secuencia infinita sin límite | Corta siempre con `take`, `limit` o `islice` antes de convertir a lista |
| Modificar una lista que otra parte del programa también usa | Devuelve una copia nueva con el cambio |
| Mezclar funciones puras con efectos secundarios | Separa el cálculo (puro) de lo que imprime o guarda |

## Para practicar

Los ejercicios [A1.5 y A1.6](ejercicios.md) usan una tubería y una secuencia perezosa. Para ver estas ideas en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
