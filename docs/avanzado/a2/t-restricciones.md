# A2.B Restricciones y variación

## Restringir el tipo

Si una función genérica necesita **hacer algo** con sus valores, como compararlos o sumarlos, tiene que saber que existe esa operación. Se lo cuentas con una **restricción**.

En Dart se restringe con **`extends`**: `T extends Comparable<T>` acepta solo tipos que se pueden comparar entre sí (`int`, `double`, `String`...), y `T extends num`, solo números. Gracias a eso, dentro de la función se puede llamar a `compareTo`. Sin restricción, de `T` no se sabe nada y no se puede llamar a ningún método suyo.

```dart
// Restricciones: «T extends ...» limita qué tipos se aceptan, y así se pueden usar sus métodos.

// Solo tipos que se pueden comparar entre sí (int, double, String...): por eso existe compareTo
T maximo<T extends Comparable<T>>(List<T> lista) {
  var mayor = lista.first;
  for (final x in lista) {
    if (x.compareTo(mayor) > 0) mayor = x;
  }
  return mayor;
}

// Solo números (int o double)
double sumar<T extends num>(List<T> lista) => lista.fold(0.0, (suma, n) => suma + n);

// Sin restricción: acepta cualquier T y una función que lo examina
int contarSi<T>(List<T> lista, bool Function(T) cumple) => lista.where(cumple).length;

void main() {
  print('maximo de [3, 9, 4]: ${maximo([3, 9, 4])}');
  print('maximo de [pera, manzana, uva]: ${maximo(['pera', 'manzana', 'uva'])}');
  print('suma de [1, 2, 3]: ${sumar([1, 2, 3])}');
  print('suma de [0.5, 0.25]: ${sumar([0.5, 0.25])}');
  print('pares en [1..6]: ${contarSi([1, 2, 3, 4, 5, 6], (n) => n.isEven)}');
  // maximo([Object(), Object()]) no compila: Object no es Comparable
}
```

Salida:

```text
maximo de [3, 9, 4]: 9
maximo de [pera, manzana, uva]: uva
suma de [1, 2, 3]: 6.0
suma de [0.5, 0.25]: 0.75
pares en [1..6]: 3
```

| Función | Qué demuestra |
|---|---|
| `maximo` | Funciona con enteros **y** con textos, pero solo con tipos que se pueden comparar |
| `sumar` | Una restricción a **números**: acepta enteros y decimales |
| `contarSi` | **Sin** restricción: no necesita saber nada de `T`, porque delega en la función que recibe |

## ¿Una lista de enteros es una lista de números?

Parece que sí, pero depende de si la lista se puede **modificar**. Cada lenguaje lo resuelve distinto:

En Dart los genéricos son **covariantes**: una `List<int>` se puede usar donde se espera una `List<num>`. Es cómodo, pero tiene una trampa: si luego intentas añadir un `double` a esa lista, Dart lanza un error **al ejecutar**, porque por dentro sigue siendo una lista de enteros.

!!! tip "Cómo decidir en Dart"
    Pregúntate si tu función **solo lee** de la colección, solo **escribe** o hace las dos cosas. Si solo lee, acepta el tipo más amplio que puedas; si hace las dos, el tipo tiene que ser exacto.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Restringir demasiado (por ejemplo, pedir `ArrayList` cuando vale cualquier lista) | Pide la interfaz más general que necesites |
| Llamar a un método que la restricción no garantiza | Añade la restricción que lo garantice, o recibe una función como parámetro |
| Querer escribir en una colección que se ha recibido como «solo lectura» | Recibe una colección modificable del tipo exacto |

## Para practicar

Los ejercicios [A2.3 y A2.5](ejercicios.md) usan una restricción y una función como criterio. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
