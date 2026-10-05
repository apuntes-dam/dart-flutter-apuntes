# Sesión 3 · Asincronía y primera app

Tras ver `Future`, `async`/`await` y crear la primera app.

## R17

**La cafetera.** Función `prepararCafe()` que devuelve un `Future<String>` y tarda 3 s. En `main`: muestra `Enciendo la cafetera`, espera el café y muestra `¡Café listo!`.

<details class="sol" data-key="sesion3/R17">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Future&lt;String&gt; prepararCafe() async {
  await Future.delayed(const Duration(seconds: 3));
  return 'Café';
}
Future&lt;void&gt; main() async {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## R18

**Adivina el orden.** Sin ejecutarlo, escribe qué imprime este código y explica por qué:

```dart
Future<void> main() async {
  print('A');
  Future.delayed(Duration.zero, () => print('B'));
  print('C');
  await Future.delayed(const Duration(milliseconds: 10));
  print('D');
}
```

## R19

**Tres descargas.** Tres tareas que tardan 1, 2 y 3 s. Mide el tiempo pidiéndolas una detrás de otra y después todas a la vez.

<details class="sol" data-key="sesion3/R19">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Future&lt;String&gt; descargar(String nombre, int segundos) =&gt;
    Future.delayed(Duration(seconds: segundos), () =&gt; nombre);
Future&lt;void&gt; main() async {
  // Una detrás de otra
  var reloj = Stopwatch()..start();
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## R20

**División peligrosa.** Función asíncrona `dividir(a, b)` que tarda 1 s y lanza una excepción si `b` es 0. Captúrala con `try`/`catch`.

## R21

**Cuenta atrás.** Un `Stream<int>` con `async*` que cuente de 10 a 0 cada medio segundo. Consúmelo con `await for` mostrando solo los pares.

## R22

**Contador que resta.** En `hola_mundo`, añade un segundo botón que reste y haz que el número se ponga en rojo cuando sea negativo. Usa solo *hot reload*.
