# Sesión 1 · Primeros pasos

Ejercicios cortos (5-10 minutos) para coger soltura con variables, operadores, cadenas y null safety. Cada uno en su propio archivo.

## R1

**Hola.** Muestra tu nombre, tu ciclo y tu ciudad en tres líneas. Después guarda esos datos en variables y muéstralos en una sola línea con interpolación: `Soy Lucía, estudio 2º DAM en Cádiz`.

## R2

**Hola, mundo en varias plataformas.**

1. Crea un proyecto: `flutter create --org com.ejemplo hola_mundo`.
2. En `lib/main.dart`, cambia el título de la pantalla por `Hola, <tu nombre>`.
3. Ejecútalo en Chrome (`flutter run -d chrome`) y en otra plataforma (emulador o escritorio). Haz una captura de cada una.
4. Con la app abierta, vuelve a cambiar el texto y pulsa `r`. ¿Qué ha pasado con el contador?

## R3

**Ticket del bar.** Con las variables `producto` (texto), `precio` (decimal), `unidades` (entero) y `terraza` (booleano), calcula el total (si es en terraza, suma un 10 %) y muéstralo así:

```text
3 x Café con leche a 1.40 € = 4.62 € (terraza)
```

Redondea a 2 decimales. Con `terraza = false` no debe aparecer el paréntesis.

<details class="sol" data-key="sesion1/R3">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>void main() {
  const producto = 'Café con leche';
  const precio = 1.40;
  const unidades = 3;
  const terraza = true;
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## R4

**Conversor.**

1. Pasa 25 °C, 0 °C y 40 °C a Fahrenheit (`F = C · 9 / 5 + 32`).
2. Declara una constante `tasaDolar = 1.08` y convierte 50 € a dólares.
3. Responde: ¿por qué `tasaDolar` puede ser constante de compilación y la hora actual no?

## R5

**Segundos a horas.** Dado `segundos = 7384`, muestra `2 h 3 min 4 s` usando la división entera y el resto. Extra: muéstralo como `02:03:04`, rellenando con ceros.

<details class="sol" data-key="sesion1/R5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>void main() {
  const segundos = 7384;
  final horas = segundos ~/ 3600;
  final minutos = (segundos % 3600) ~/ 60;
  final resto = segundos % 60;
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## R6

**¿Tienes apodo?** Declara `nombre = 'Francisco'` y un `String? apodo` sin valor.

1. Saluda con el apodo si lo tiene y, si no, con el nombre, en una sola línea.
2. Muestra la longitud del apodo sin que aparezca `null`.
3. Dale el valor `'Curro'` y vuelve a ejecutar. ¿Qué cambia?

## R7

**Iniciales.** Con `nombre = 'maría'`, `apellido1 = 'ruiz'` y `apellido2 = 'pérez'`:

1. Muestra las iniciales en mayúsculas: `M.R.P.`.
2. Muestra el nombre completo con la inicial de cada parte en mayúscula: `María Ruiz Pérez`.
3. Muestra cuántas letras tiene en total, sin contar espacios.

## R8

**Reto: ¿mayor de edad?** (solo en local) Pide la edad por teclado, convirtiéndola de forma segura.

* Si no es un número: `"hola" no es una edad válida`.
* Si lo es: indica si es mayor de edad y cuántos años faltan para los 18 (o cuántos han pasado).
