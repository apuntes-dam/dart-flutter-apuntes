# U7.5 · Ficheros con dart:io, bytes y bloques

<div class="ej-gate" data-unit="u07" data-nombre="U7 · Entrada/salida y GUI"></div>

Ejercicios de [7.E](t-path.md): `File`, `Directory`, ficheros binarios y lectura y escritura por bloques. Todos controlan los errores con `try`/`on … catch` y cierran los archivos con `closeSync()` en un `finally`.

## Ejercicio 7.16

**Gestor de archivos con `dart:io`.** Haz un menú: 1 crear carpeta, 2 crear archivo vacío, 3 ver información (ruta absoluta, nombre, carpeta padre, si es archivo o carpeta y tamaño), 4 copiar, 5 mover o renombrar, 6 borrar y 7 salir. Cada opción pide los nombres que necesite. Controla los errores con `try`/`on … catch` y muestra un mensaje distinto para `PathExistsException`, `PathNotFoundException` y `FileSystemException`. Recuerda que `createSync()` de una carpeta, `copySync` y `renameSync` **no fallan** si el destino ya existe: antes de usarlos, comprueba tú el destino con `existsSync()` y avisa.

<details class="sol" data-key="kx/u7-5/7.16">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import 'dart:io';
String pedir(String texto) {
  stdout.write(texto);
  return (stdin.readLineSync() ?? '').trim();
}
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.17

**Fichero binario de caracteres.** Menú: 1 crear, 2 mostrar y 3 salir. Al crear, pide el nombre del fichero (si existe, pregunta si se crea de nuevo) y después datos hasta que se escriba `fin`; de cada dato se guarda en el fichero, como **un byte**, el código de su primer carácter (`runes.first`). Si el código es mayor que 255 (como el de `€`), avisa de que no cabe en un byte y no lo guardes. Al mostrar, lee el fichero byte a byte y escribe, por cada uno, su número y su carácter.

<details class="sol" data-key="kx/u7-5/7.17">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import 'dart:io';
String pedir(String texto) {
  stdout.write(texto);
  return (stdin.readLineSync() ?? '').trim();
}
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.18

**Copiar un binario por bloques.** Pide el nombre de un fichero origen y uno destino y cópialo leyendo y escribiendo **bloques de 4096 bytes** (sin `copySync` ni `readAsBytesSync`). Muestra cuántos bloques se han copiado y cuántos bytes en total, y comprueba que origen y destino pesan lo mismo.

<details class="sol" data-key="kx/u7-5/7.18">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import 'dart:io';
import 'dart:typed_data';
void main() {
  stdout.write('Origen: ');
  final origen = File((stdin.readLineSync() ?? '').trim());
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.19

**Escribir un texto por bloques.** Pide un texto y un tamaño de bloque. Escríbelo en un fichero **por trozos de ese tamaño**, cortando el texto con `substring(inicio, fin)` y escribiendo cada trozo con `writeStringSync`. Muestra cuántas escrituras han hecho falta y el contenido final del fichero.

<details class="sol" data-key="kx/u7-5/7.19">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import 'dart:io';
void main() {
  stdout.write('Texto: ');
  final texto = stdin.readLineSync() ?? '';
  stdout.write('Tamaño del bloque: ');
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.20

**Contar leyendo por trozos.** Pide el nombre de un fichero de texto y léelo con `openRead().transform(utf8.decoder)` (Dart decide el tamaño de cada trozo). Muestra cuántos trozos se han leído, cuántas vocales y cuántas líneas tiene. Pruébalo con un fichero de más de 64 KB para ver más de un trozo, y cuida la última línea, que puede no acabar en salto de línea.

<details class="sol" data-key="kx/u7-5/7.20">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import 'dart:convert';
import 'dart:io';
Future&lt;void&gt; main() async {
  stdout.write('Fichero: ');
  final ruta = (stdin.readLineSync() ?? '').trim();
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.21

**Caracteres y bytes.** Pide un texto, guárdalo en un fichero y muestra cuántos caracteres tiene (`length` y `runes.length`) y cuántos **bytes** ocupa en el disco. Pruébalo con `hola` y con `Año 5 €` y explica por qué en el segundo caso los números no coinciden.

<details class="sol" data-key="kx/u7-5/7.21">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import 'dart:io';
void main() {
  stdout.write('Texto: ');
  final texto = stdin.readLineSync() ?? '';
  final ruta = File('texto.txt');
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
