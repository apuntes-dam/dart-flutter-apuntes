# A1.A Funciones como valores

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5 (variables, estructuras de control, colecciones y clases). Si alguna idea se te resiste, vuelve a la unidad correspondiente.

Hasta ahora has **llamado** funciones. Aquí das un paso más: tratar una función **como un valor**. Se puede guardar en una variable, pasar a otra función y devolver desde una función. Es la base de cosas que ya usas sin darte cuenta: ordenar con un criterio, reaccionar a un clic o filtrar una lista.

## Tipo de una función en Dart

En Dart cada función tiene un tipo: `int Function(int)` es «una función que recibe un `int` y devuelve un `int`». Una función con nombre se pasa tal cual (`doble`, sin paréntesis) y una anónima se escribe `(n) => n * 2` o `(n) { ... }`.

## Un ejemplo con todo

El programa define una función, la pasa a otra, devuelve una función desde una función y crea dos contadores independientes:

```dart
// Funciones como valores: guardarlas, pasarlas, devolverlas y capturar variables.
int doble(int n) => n * 2;

// Recibe una función como parámetro
int aplicarDosVeces(int Function(int) f, int x) => f(f(x));

// Devuelve una función que "recuerda" factor
int Function(int) multiplicador(int factor) => (n) => n * factor;

// Cierre: la función devuelta recuerda y modifica 'cuenta'
int Function() crearContador() {
  var cuenta = 0;
  return () => ++cuenta;
}

void main() {
  final cuadrado = (int n) => n * n; // función anónima guardada en una variable

  print('doble(4) = ${doble(4)}');
  print('cuadrado(4) = ${cuadrado(4)}');
  print('aplicar dos veces doble a 3 = ${aplicarDosVeces(doble, 3)}');
  print('multiplicador(5)(7) = ${multiplicador(5)(7)}');

  final a = crearContador();
  final b = crearContador();
  print('contador A: ${a()} ${a()} ${a()}');
  print('contador B: ${b()}');
}
```

Salida:

```text
doble(4) = 8
cuadrado(4) = 16
aplicar dos veces doble a 3 = 12
multiplicador(5)(7) = 35
contador A: 1 2 3
contador B: 1
```

Qué ocurre en cada línea:

| Línea de salida | Qué demuestra |
|---|---|
| `doble(4)` y `cuadrado(4)` | Una función con nombre y otra **anónima guardada en una variable** se llaman igual |
| `aplicar dos veces doble a 3` | Una función recibe **otra función como parámetro** y la usa dos veces |
| `multiplicador(5)(7)` | Una función **devuelve otra función**, que recuerda el factor `5` |
| `contador A` y `contador B` | Cada contador **recuerda su propia cuenta**: no se pisan |

## Cierres: funciones que recuerdan

Los **cierres** (*closures*) de Dart capturan las variables, no solo sus valores: `crearContador` devuelve una función que sigue viendo y modificando `cuenta` aunque `crearContador` ya haya terminado, y cada llamada a `crearContador` crea su propia `cuenta`.

!!! tip "Cuándo usarlo"
    Si necesitas «una operación que se decide más tarde» (un criterio de orden, un filtro, qué hacer al terminar), pásala como función. Es más corto y más flexible que crear una clase solo para eso.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a la función al pasarla (`doble(3)` en vez de `doble`) | Pasa el nombre **sin paréntesis** si quieres pasar la función, no su resultado |
| Esperar que cada llamada comparta o no comparta estado sin comprobarlo | Crea el cierre una vez por contador, como en el ejemplo |
| Escribir lambdas largas e ilegibles | Si pasa de una o dos líneas, ponle nombre a la función |

## Para practicar

Los ejercicios [A1.1 y A1.2](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
