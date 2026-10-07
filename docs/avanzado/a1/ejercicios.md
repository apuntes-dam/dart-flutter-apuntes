# A1 · Ejercicios de programación funcional

<div class="ej-gate" data-unit="a1" data-nombre="A1 · Programación funcional"></div>

Practica funciones como valores, colecciones sin bucles y secuencias perezosas. Cada ejercicio indica la **salida esperada** para que compruebes tu programa.

## Ejercicio A1.1

**Aplicar una función varias veces.** Escribe `aplicarNVeces(f, n, x)`, que aplica la función `f` al valor `x`, `n` veces seguidas (con `n = 0` devuelve `x` sin cambios).

En `main` pruébala con estas llamadas, cada resultado en su línea:

* «doble» (multiplicar por 2) aplicada 3 veces a `1`.
* «sumar 3» aplicada 4 veces a `0`.
* «doble» aplicada 0 veces a `1`.

**Salida esperada:**

```text
8
12
1
```

!!! note "En Dart"
    El tipo de un parámetro que es una función de `int` a `int` se escribe `int Function(int)`.

<details class="sol" data-key="av/a1/A1.1">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>int aplicarNVeces(int Function(int) f, int n, int x) {
  var resultado = x;
  for (var i = 0; i &lt; n; i++) {
    resultado = f(resultado);
  }
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A1.2

**Contador con paso.** Escribe `crearContador(inicio, paso)`, que devuelve una función. Cada vez que se llama a esa función devuelve el valor actual y después avanza `paso` unidades.

En `main` crea un contador que empieza en `10` y avanza de `5` en `5`, y otro que empieza en `0` y avanza de `1` en `1`. Llama tres veces al primero (en una sola línea, separando con espacios) y una vez al segundo. Comprueba que **cada contador lleva su propia cuenta**.

**Salida esperada:**

```text
10 15 20
0
```

<details class="sol" data-key="av/a1/A1.2">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>int Function() crearContador(int inicio, int paso) {
  var actual = inicio;
  return () {
    final valor = actual;
    actual += paso;
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A1.3

**Palabras seleccionadas.** Dada la lista `sol, luna, estrella, mar, cielo, nube`:

1. Quédate con las palabras de **4 letras o más**.
2. Pásalas a **mayúsculas**.
3. **Ordénalas** alfabéticamente.
4. Muestra el resultado unido con guiones y, en otra línea, cuántas palabras quedaron.

Hazlo **sin escribir un bucle**: encadena operaciones de colección.

**Salida esperada:**

```text
CIELO-ESTRELLA-LUNA-NUBE
4 palabras
```

<details class="sol" data-key="av/a1/A1.3">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>void main() {
  final palabras = ['sol', 'luna', 'estrella', 'mar', 'cielo', 'nube'];
  final resultado = palabras
      .where((p) =&gt; p.length &gt;= 4)
      .map((p) =&gt; p.toUpperCase())
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A1.4

**Contar por longitud.** Con la misma lista del ejercicio anterior (`sol, luna, estrella, mar, cielo, nube`), cuenta **cuántas palabras hay de cada longitud** y muestra el resultado ordenado por longitud, con el formato `longitud:cantidad` separado por comas y espacios.

**Salida esperada:**

```text
3:2, 4:2, 5:1, 8:1
```

!!! note "En Dart"
    Dart no trae un `groupBy` en la biblioteca estándar: usa `fold` sobre un mapa o un bucle corto.

<details class="sol" data-key="av/a1/A1.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>void main() {
  final palabras = ['sol', 'luna', 'estrella', 'mar', 'cielo', 'nube'];
  final cuenta = palabras.fold&lt;Map&lt;int, int&gt;&gt;({}, (mapa, p) {
    mapa[p.length] = (mapa[p.length] ?? 0) + 1;
    return mapa;
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A1.5

**Una tubería de funciones.** Escribe `pipeline(funciones)`, que recibe una lista de funciones de entero a entero y devuelve **una sola función** que las aplica una tras otra, en orden.

En `main`: construye la tubería «multiplicar por 2», «sumar 1», «elevar al cuadrado» y aplícala a `5`. Después comprueba qué ocurre con una tubería **vacía** aplicada a `7`.

**Salida esperada:**

```text
121
7
```

<details class="sol" data-key="av/a1/A1.5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>int Function(int) pipeline(List&lt;int Function(int)&gt; funciones) =&gt;
    (x) =&gt; funciones.fold(x, (acumulado, f) =&gt; f(acumulado));
void main() {
  final tuberia = pipeline([
    (n) =&gt; n * 2,
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A1.6

**Secuencia perezosa.** Obtén los **cuatro primeros** números naturales (empezando en 1) que sean múltiplos de 7 y mayores que 20, y muéstralos como una lista.

La secuencia de números naturales es **infinita**: usa una secuencia perezosa y pide solo los cuatro que necesitas.

**Salida esperada:**

```text
[21, 28, 35, 42]
```

!!! note "En Dart"
    Un generador `sync*` con `yield` produce valores sin límite; `take(4)` pide solo cuatro.

<details class="sol" data-key="av/a1/A1.6">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Iterable&lt;int&gt; naturales() sync* {
  var n = 1;
  while (true) {
    yield n++;
  }
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
