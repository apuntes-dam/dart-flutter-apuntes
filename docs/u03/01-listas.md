# 3.1 Listas y tuplas

Una **lista** guarda varios valores **ordenados** en una sola variable, y se accede a cada uno por su posición (empezando en 0). Puede crecer y encogerse.

En Dart la lista es `List<T>`. La literal `[1, 2, 3]` es **modificable**; con `const [1, 2, 3]` o `List.unmodifiable(...)` es de solo lectura. `final lista = [..]` impide **reasignar la variable**, pero sí se puede cambiar su contenido.

## Operaciones básicas

```dart
void main() {
  final lista = [5, 3, 8, 1];
  print('lista: $lista');
  print('tamaño: ${lista.length}, primero: ${lista.first}, último: ${lista.last}');

  lista.add(9);
  print('tras añadir 9: $lista');
  lista.insert(1, 7);
  print('insertar 7 en la posición 1: $lista');
  lista.remove(3);
  print('quitar el valor 3: $lista');
  lista.removeAt(0);
  print('quitar la posición 0: $lista');
  lista.sort();
  print('ordenada: $lista');

  var suma = 0;
  for (final n in lista) {
    suma += n;
  }
  print('suma: $suma, mayor: ${lista.reduce((a, b) => a > b ? a : b)}');
}
```

Salida:

```text
lista: [5, 3, 8, 1]
tamaño: 4, primero: 5, último: 1
tras añadir 9: [5, 3, 8, 1, 9]
insertar 7 en la posición 1: [5, 7, 3, 8, 1, 9]
quitar el valor 3: [5, 7, 8, 1, 9]
quitar la posición 0: [7, 8, 1, 9]
ordenada: [1, 7, 8, 9]
suma: 25, mayor: 9
```

| Operación | En Dart |
|---|---|
| Añadir al final | `lista.add(9)` |
| Insertar en una posición | `lista.insert(1, 7)` |
| Quitar un **valor** | `lista.remove(3)` |
| Quitar una **posición** | `lista.removeAt(0)` |
| Ordenar (modifica la lista) | `lista.sort()` |
| Posición de un valor | `lista.indexOf(8)` (`-1` si no está) |
| ¿Contiene? | `lista.contains(8)` |
| Tamaño · primero · último | `length` · `first` · `last` |
| Una parte | `lista.sublist(1, 3)` |
| Vaciar | `lista.clear()` |

No hay confusión posible: `remove` quita un valor y `removeAt` una posición.

!!! warning "No cambies una lista mientras la recorres"
    Añadir o quitar elementos dentro del bucle que la recorre da errores o resultados raros. Si quieres filtrar, construye una **lista nueva** con lo que sirve, o usa el método específico de borrado por condición de tu lenguaje.

## Recorrer, copiar y listas anidadas

```dart
void main() {
  final numeros = [10, 20, 30];
  final partes = <String>[];
  for (var i = 0; i < numeros.length; i++) {
    partes.add('$i:${numeros[i]}');
  }
  print('con índice: ${partes.join(' ')}');

  final alias = numeros;
  alias[0] = 99;
  print('alias: $numeros');

  final copia = List.of(numeros);
  copia[0] = 1;
  print('original: $numeros, copia: $copia');

  final matriz = [
    [1, 2, 3],
    [4, 5, 6],
  ];
  for (final fila in matriz) {
    print('fila: $fila');
  }
  print('elemento [1][2] = ${matriz[1][2]}');
}
```

Salida:

```text
con índice: 0:10 1:20 2:30
alias: [99, 20, 30]
original: [99, 20, 30], copia: [1, 20, 30]
fila: [1, 2, 3]
fila: [4, 5, 6]
elemento [1][2] = 6
```

Hay dos ideas importantes en este ejemplo:

* **Una lista es una referencia.** `alias = numeros` no crea otra lista: las dos variables apuntan a la **misma**, y al cambiar una cambia la otra. Para tener una lista independiente hay que **copiarla**: `final copia = List.of(numeros)` (o `[...numeros]`).
* **La copia es superficial.** Copia los elementos, pero si estos son a su vez listas (como en una matriz), las sublistas se **comparten** entre original y copia.

Una **lista de listas** sirve como tabla o matriz: `matriz[fila][columna]`. Se recorre con dos bucles, uno dentro de otro.

## Tuplas y transformar listas

```dart
void main() {
  final persona = ('Ana', 25);
  print('${persona.$1} tiene ${persona.$2} años');
  final (nombre, edad) = persona;
  print('desempaquetado: $nombre, $edad');

  final numeros = [1, 2, 3, 4, 5, 6];
  final cuadrados = [
    for (final n in numeros)
      if (n.isEven) n * n,
  ];
  print('cuadrados de los pares: $cuadrados');
  final suma = cuadrados.fold(0, (a, b) => a + b);
  print('suma de los cuadrados: $suma');
}
```

Salida:

```text
Ana tiene 25 años
desempaquetado: Ana, 25
cuadrados de los pares: [4, 16, 36]
suma de los cuadrados: 56
```

Una **tupla** agrupa un número fijo de valores, quizá de distinto tipo, sin crear una clase. En Dart 3 son los **records**: `('Ana', 25)` se accede con `$1` y `$2`, o se **desempaqueta** con `final (nombre, edad) = persona;`.

En Dart, `for` dentro de una lista literal con `if` (como en el ejemplo) o los métodos `where` (filtrar), `map` (transformar) y `fold`/`reduce` (acumular).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Acceder a una posición que no existe | Comprueba el tamaño antes: la última posición es el tamaño menos 1 |
| Creer que `b = a` copia la lista | Copia explícitamente cuando necesites una independiente |
| Ordenar una lista y perder el orden original | Ordena una **copia**, o usa la versión que devuelve una lista nueva |
| Modificar la lista mientras se recorre | Construye una lista nueva con el resultado |

## Para practicar

Haz los [ejercicios 3.1 de listas](listas.md). Compara cómo se escribe lo mismo en otro lenguaje en [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
