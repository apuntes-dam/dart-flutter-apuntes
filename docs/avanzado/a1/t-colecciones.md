# A1.B Colecciones sin bucles

Casi todo lo que haces con una lista con un `for` entra en unas pocas operaciones: **quedarse con** unos elementos, **transformarlos**, **reducirlos** a un valor, **ordenarlos**, **agruparlos** y **preguntar** si alguno o todos cumplen algo. Si las pasas como funciones, el programa dice **qué** quieres, no **cómo** recorrerlo.

## Las operaciones en Dart

| Qué quieres | Cómo se hace |
|---|---|
| Quedarse con los que cumplen | `where((p) => ...)` |
| Transformar cada elemento | `map((p) => ...)` |
| Reducir a un solo valor | `fold(inicial, (acum, x) => ...)` o `reduce` |
| Ordenar con un criterio | `sort((a, b) => ...)` (**modifica** la lista) |
| Agrupar o contar por clave | `fold` sobre un `Map` (no hay `groupBy` listo) |
| ¿Alguno / todos? | `any(...)` / `every(...)` |

## Un ejemplo completo

Una pastelería guarda sus pedidos (producto, cantidad y precio por unidad). Con las operaciones anteriores se responde a todo sin un solo bucle explícito:

```dart
// Pedidos de una pastelería: filtrar, transformar, reducir, ordenar y agrupar sin escribir bucles.
typedef Pedido = ({String producto, int cantidad, int precio});

int importe(Pedido p) => p.cantidad * p.precio;

void main() {
  final List<Pedido> pedidos = [
    (producto: 'tarta', cantidad: 2, precio: 18),
    (producto: 'galleta', cantidad: 12, precio: 1),
    (producto: 'tarta', cantidad: 1, precio: 18),
    (producto: 'pan', cantidad: 3, precio: 2),
    (producto: 'galleta', cantidad: 7, precio: 1),
  ];

  // filter + map: los productos de los pedidos con 3 o más unidades
  final grandes = pedidos.where((p) => p.cantidad >= 3).map((p) => p.producto);
  print('grandes: ${grandes.join(", ")}');

  // map: de pedidos a importes
  final importes = pedidos.map(importe).toList();
  print('importes: $importes');

  // reduce / fold: de muchos valores a uno
  print('total: ${importes.fold(0, (suma, i) => suma + i)}');

  // sort con un criterio (sobre una copia: sort modifica la lista)
  final ordenados = [...pedidos]..sort((a, b) => importe(b).compareTo(importe(a)));
  print('mayor a menor: ${ordenados.map((p) => "${p.producto} ${importe(p)}").join(", ")}');

  // agrupar: unidades por producto
  final porProducto = pedidos.fold<Map<String, int>>({}, (mapa, p) {
    mapa[p.producto] = (mapa[p.producto] ?? 0) + p.cantidad;
    return mapa;
  });
  final claves = porProducto.keys.toList()..sort();
  print('unidades: ${claves.map((k) => "$k=${porProducto[k]}").join(", ")}');

  // any / every: preguntas sobre toda la colección
  print('¿alguno vale más de 30? ${pedidos.any((p) => importe(p) > 30) ? "sí" : "no"}');
  print('¿todos tienen cantidad > 0? ${pedidos.every((p) => p.cantidad > 0) ? "sí" : "no"}');
}
```

Salida:

```text
grandes: galleta, pan, galleta
importes: [36, 12, 18, 6, 7]
total: 79
mayor a menor: tarta 36, tarta 18, galleta 12, galleta 7, pan 6
unidades: galleta=19, pan=3, tarta=3
¿alguno vale más de 30? sí
¿todos tienen cantidad > 0? sí
```

Cada línea de la salida es una de las operaciones de la tabla. Compruébalas a mano: los importes son `2·18`, `12·1`, `1·18`, `3·2` y `7·1`; el total es `79`; y hay `19` galletas porque `12 + 7`.

## Lo que conviene saber

`where` y `map` devuelven un `Iterable` **perezoso**: no hacen nada hasta que lo recorres o llamas a `toList()`. Y `sort` **cambia la lista original**, por eso el ejemplo ordena una copia (`[...pedidos]..sort(...)`).

!!! warning "Un orden estable"
    Si dos elementos empatan en el criterio de orden, no des por hecho en qué orden saldrán. Los datos del ejemplo no tienen empates a propósito. Si te hacen falta, añade un segundo criterio.

!!! tip "¿Bucle o función?"
    No es mejor siempre una cosa que otra. Un bucle sigue siendo lo más claro cuando el cuerpo hace varias cosas distintas o hay que parar a mitad. Usa las operaciones de colección cuando el trabajo es **una transformación de datos** y verás que el código se lee casi como la descripción del problema.

## Para practicar

Los ejercicios [A1.3 y A1.4](ejercicios.md) usan estas operaciones. Para ver la misma tabla en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
