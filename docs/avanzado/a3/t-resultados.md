# A3.B Resultados, errores y esperas

Lanzar una tarea es solo la mitad: casi siempre querrás **recoger su resultado** y saber si **falló**. Lo que representa «un resultado que todavía no existe» recibe distintos nombres según el lenguaje, pero la idea es la misma.

## Resultados futuros en Dart

Una función `async` devuelve un **`Future<T>`**: una promesa de un `T` que llegará (o de un error). `await` espera ese valor **sin bloquear**. `Future.wait([...])` espera a varios y devuelve una lista con los resultados **en el mismo orden**. Para limitar la espera: `future.timeout(Duration(...))`, que lanza `TimeoutException`.

| Qué | En Dart |
|---|---|
| Un valor que llegará | `Future<T>` |
| Esperar el valor | `await f` |
| Lanzar varias y esperar todas | `Future.wait` |
| Límite de tiempo | `.timeout(...)` |

## Un ejemplo: tres consultas y un error

Se consultan los precios de tres productos **a la vez** (cada consulta tarda 100 ms), se suman, y después se pide un producto que no existe para ver cómo llega el error:

```dart
// Tareas que DEVUELVEN un resultado (o un error): se lanzan todas a la vez y se recogen con await.
const precios = {'pan': 2, 'leche': 1, 'queso': 5};

Future<int> precio(String producto) async {
  await Future.delayed(const Duration(milliseconds: 100)); // simula una consulta lenta
  final p = precios[producto];
  if (p == null) throw ArgumentError('producto desconocido: $producto');
  return p;
}

Future<void> main() async {
  final productos = ['pan', 'leche', 'queso'];

  final consultas = productos.map(precio).toList(); // las tres consultas avanzan a la vez
  final resultados = await Future.wait(consultas); // una lista con los resultados, en el mismo orden
  for (var i = 0; i < productos.length; i++) {
    print('precio de ${productos[i]}: ${resultados[i]}');
  }
  print('total: ${resultados.reduce((a, b) => a + b)}');

  // un error dentro de una tarea se lanza al hacer await, y se captura con try/catch como siempre
  try {
    await precio('caviar');
  } on ArgumentError catch (e) {
    print('error: ${e.message}');
  }
}
```

Salida:

```text
precio de pan: 2
precio de leche: 1
precio de queso: 5
total: 8
error: producto desconocido: caviar
```

El error de una tarea se lanza al hacer `await`, como una excepción normal: se captura con `try`/`on`.

Los resultados se muestran **en el orden en que se pidieron**, aunque las consultas hayan terminado en otro. Es lo que se espera de una lista de resultados.

!!! tip "Esperar no es lo mismo que bloquear"
    `await`, `join` o `get` **esperan** al resultado. En los modelos con `await` (Dart, Kotlin, Python) esa espera deja libre al hilo para otras tareas. En `join()`/`get()` de Java **el hilo que espera se queda parado**: es lo normal en un programa de consola, pero no en una interfaz gráfica, que se congelaría.

## Límites de tiempo y cancelación

Una tarea que tarda demasiado puede dejar tu programa colgado. Todo lenguaje ofrece una **espera con límite** (mira la tabla de arriba): si se pasa el tiempo, obtienes un error o un valor vacío y decides qué hacer, por ejemplo **reintentar** o avisar. Cancelar la tarea libera recursos, pero en muchos casos solo se **solicita** y la tarea tiene que cooperar.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Lanzar varias tareas y esperarlas **una por una al lanzarlas** (cada una espera a la anterior) | Lánzalas todas primero y luego espera: así avanzan a la vez |
| Olvidar capturar el error de una tarea | El error llega al esperar el resultado: ahí va el `try` |
| Esperar sin límite una operación de red | Pon un límite de tiempo |
| Reintentar sin pausa ni tope | Limita el número de intentos (ejercicio [A3.4](ejercicios.md)) y espera entre ellos |

## Para practicar

Los ejercicios [A3.3 y A3.4](ejercicios.md) usan un límite de tiempo y reintentos. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
