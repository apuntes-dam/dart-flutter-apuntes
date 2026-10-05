# Boletín de ejercicios

Individual. Cada ejercicio en su propio archivo (`e1.dart`, `e2.dart`…). Ejecuta con `dart run eN.dart`. Tiempo orientativo: 20-30 minutos por ejercicio.

## E1

**Diagnóstico del entorno.**

1. Ejecuta `flutter doctor -v` y guarda una captura.
2. Crea `e1.md` con una tabla: cada línea del doctor, su estado (✓, ! o ✗) y si hay que arreglarla para compilar a Android y web. Justifícalo en una frase.
3. Ejecuta `flutter devices` y copia la salida. ¿Cuántas plataformas puedes usar ahora? ¿Cuál te falta y qué tendrías que instalar?

## E2

**Ficha de alumno (tipos y null safety).**

1. Declara las variables de un alumno: `nombre` (String), `edad` (int), `notaMedia` (double), `repetidor` (bool), `segundoApellido` (String?) y `email` (String?, sin inicializar).
2. Muestra una ficha con interpolación. Si no hay segundo apellido, no debe aparecer `null`.
3. Si `email` es nulo, muestra `sin email`.
4. Asigna un email solo si es nulo y muestra su longitud sin usar `!`.
5. (Solo en local) Pide la edad por teclado y controla que sea un número.

## E3

**Calculadora de notas (control de flujo y funciones).**

1. Escribe `String calificacion(double nota)` con una selección múltiple que devuelva `Insuficiente`, `Suficiente`, `Bien`, `Notable`, `Sobresaliente` o `Matrícula`. Si la nota no está entre 0 y 10, devuelve `Nota no válida`.
2. Escribe `double media(List<double> notas, {bool redondear = false})`. Si `redondear` es verdadero, devuelve la media con un decimal.
3. En `main`, recorre `[3.5, 5, 6.8, 9.2, 10, 11]`, muestra cada nota con su calificación y después la media con y sin redondeo.

## E4

**La cesta del mercado (colecciones).** Dado este mapa de precios por kilo:

```dart
final precios = {
  'boquerones': 7.90,
  'cazón': 12.50,
  'chocos': 11.00,
  'tomates': 2.30,
  'pimientos': 2.80,
};
```

1. Muestra solo los productos de menos de 5 €/kg.
2. Crea una lista con los nombres en mayúsculas.
3. Dada una cesta `{'boquerones': 0.5, 'tomates': 1.2, 'chocos': 0.75}` (producto → kilos), calcula el total y muéstralo con dos decimales.
4. Añade a la cesta un producto que no esté en `precios` y haz que tu código lo ignore sin romperse.

## E5

**Agrupaciones del COAC (clases y enumerados).**

1. Crea el enumerado `Modalidad` (comparsa, chirigota, coro, cuarteto) con una etiqueta legible.
2. Crea la clase `Agrupacion` con un constructor de parámetros con nombre: `nombre` y `modalidad` obligatorios, `autor` opcional y `puntos` con valor por defecto 0. Añade un getter `finalista` (más de 300 puntos) y un `toString` que muestre `autor desconocido` si no hay autor.
3. En `main`, crea una lista de 6 agrupaciones y muestra: las finalistas, la de más puntos y cuántas hay de cada modalidad.

## E6

**Descargas simuladas (asincronía).**

1. Escribe `Future<List<String>> descargarAgrupaciones()` que tarde 2 segundos y devuelva una lista de nombres.
2. Escribe `Future<String> descargarLetra(String agrupacion)` que tarde entre 1 y 3 segundos (usa un aleatorio) y falle si el nombre contiene `'X'`.
3. En `main`: descarga la lista; después descarga las letras de todas a la vez, captura los errores y muestra el tiempo total con un cronómetro.
4. Responde en un comentario: ¿cuánto tardaría si las descargaras una detrás de otra? ¿Por qué?
