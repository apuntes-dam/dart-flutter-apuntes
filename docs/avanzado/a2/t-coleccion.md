# A2.C Tu colección y qué pasa al ejecutar

## Una colección genérica propia

Las listas, mapas y conjuntos que ya usas son clases genéricas. Aquí escribimos una a mano: una **pila**, donde el último elemento en entrar es el primero en salir (como una pila de platos). La misma clase sirve para letras y para enteros:

```dart
// Una colección genérica propia: una pila (el último en entrar es el primero en salir).
class Pila<T> {
  final List<T> _elementos = [];

  void apilar(T x) => _elementos.add(x);

  T desapilar() {
    if (_elementos.isEmpty) throw StateError('la pila está vacía');
    return _elementos.removeLast();
  }

  T get tope => _elementos.last;
  int get tamano => _elementos.length;
  bool get vacia => _elementos.isEmpty;
}

void main() {
  final letras = Pila<String>();
  for (final l in ['a', 'b', 'c']) {
    letras.apilar(l);
  }
  print('tamaño: ${letras.tamano}');
  print('tope: ${letras.tope}');
  print('sacar: ${letras.desapilar()}');
  print('sacar: ${letras.desapilar()}');
  print('quedan: ${letras.tamano}');
  print('vacía: ${letras.vacia ? "sí" : "no"}');
  print('sacar: ${letras.desapilar()}');
  try {
    letras.desapilar();
  } on StateError catch (e) {
    print('error: ${e.message}');
  }

  // la misma clase, con otro tipo
  final numeros = Pila<int>();
  for (final n in [1, 2, 3]) {
    numeros.apilar(n);
  }
  final salida = <int>[];
  while (!numeros.vacia) {
    salida.add(numeros.desapilar());
  }
  print('pila de enteros al vaciarla: ${salida.join(" ")}');
}
```

Salida:

```text
tamaño: 3
tope: c
sacar: c
sacar: b
quedan: 1
vacía: no
sacar: a
error: la pila está vacía
pila de enteros al vaciarla: 3 2 1
```

Qué se ve en la salida:

1. Se apilan `a`, `b`, `c`; el **tope** es la última (`c`) y se sacan en orden contrario: `c`, `b`, `a`.
2. Intentar sacar de una pila **vacía** lanza una excepción con un mensaje claro, que el programa captura. No se devuelve un valor inventado.
3. Con una `Pila<Int>`, los números `1, 2, 3` salen como `3 2 1`: es la misma clase con otro tipo.

!!! tip "Una pila para algo útil"
    Una pila sirve para **deshacer** acciones, comprobar si los paréntesis de una expresión están equilibrados o recorrer estructuras sin recursión. El ejercicio [A2.4](ejercicios.md) la usa para invertir una lista.

## Qué pasa con los tipos al ejecutar

Los genéricos se comprueban **al compilar o al analizar el código**. Lo que ocurre después, al ejecutar, **depende del lenguaje**:

En Dart los tipos genéricos **existen al ejecutar** (se dice que están *reificados*): un objeto `List<int>` sabe que es de enteros, y por eso `is List<int>` es verdadero y `is List<String>` es falso. Es una diferencia importante con Java.

```dart
// En Dart los tipos genéricos EXISTEN al ejecutar: una List<int> sabe que es de enteros.
void main() {
  final enteros = <int>[1, 2];
  final Object cosa = enteros; // perdemos el tipo en el código, pero el objeto lo conserva

  print('¿es List<int>? ${cosa is List<int>}');
  print('¿es List<String>? ${cosa is List<String>}');
  print('tipo en ejecución: ${enteros.runtimeType}');
}
```

Salida:

```text
¿es List<int>? true
¿es List<String>? false
tipo en ejecución: List<int>
```

Si necesitas comprobar tipos genéricos al ejecutar, en Dart puedes: `is List<int>`, `runtimeType`, o un `switch` con patrones.

!!! warning "Por eso no se mezclan en la misma lista sin querer"
    En los lenguajes que comprueban al compilar, intentar meter un texto en una lista de enteros ni siquiera compila. En Python, el programa se ejecuta igualmente y el fallo aparece más tarde, cuando otra parte del código espera un número.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Devolver un valor inventado al sacar de una colección vacía | Lanza una excepción con un mensaje claro o devuelve un valor opcional |
| Esperar poder preguntar `¿es una lista de enteros?` al ejecutar en todos los lenguajes | Depende del lenguaje; si lo necesitas, guarda el tipo tú mismo |
| Escribir una colección propia cuando ya existe una en la biblioteca | Usa las del lenguaje salvo que quieras aprender o necesites un comportamiento distinto |

## Para practicar

Los ejercicios [A2.4 y A2.6](ejercicios.md) usan una pila y una caché genéricas. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
