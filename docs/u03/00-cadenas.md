# 3.0 Cadenas de texto

Una **cadena** es una secuencia de caracteres: un nombre, una frase, el contenido de un archivo. Casi todo programa las maneja, y casi siempre se hace lo mismo: **recorrerlas, buscar dentro, partirlas y transformarlas**.

En Dart el texto es un `String`. Las cadenas son **inmutables**: ningún método modifica la original, todos **devuelven una nueva**.

## Recorrer y buscar

```dart
void main() {
  final texto = 'banana';
  print('longitud: ${texto.length}');
  print('primera: ${texto[0]}, última: ${texto[texto.length - 1]}');
  print('subcadena: ${texto.substring(2, 5)}');

  var cuenta = 0;
  for (var i = 0; i < texto.length; i++) {
    if (texto[i] == 'a') {
      cuenta++;
    }
  }
  print('letras a: $cuenta');
  print('posición de "na": ${texto.indexOf('na')}');
  print('contiene "nan": ${texto.contains('nan') ? 'sí' : 'no'}');

  var alReves = '';
  var j = texto.length - 1;
  while (j >= 0) {
    alReves += texto[j];
    j--;
  }
  print('al revés: $alReves');
}
```

Salida:

```text
longitud: 6
primera: b, última: a
subcadena: nan
letras a: 3
posición de "na": 2
contiene "nan": sí
al revés: ananab
```

Los índices empiezan en **0** y el último es `length - 1`. Acceder fuera de rango lanza un `RangeError`.

Fíjate en el patrón de **recorrido**: una variable que va de la primera posición a la última (o al revés) y, dentro, una pregunta sobre la letra actual. Con él se cuenta, se busca y se invierte; es la base de los ejercicios de este apartado. Las cadenas ya traen métodos que lo hacen por ti (`indexOf`/`find`, `contains`...), pero conviene saber hacerlo a mano.

!!! warning "El final de una subcadena no se incluye"
    `substring(2, 5)` (o `texto[2:5]`) toma las posiciones **2, 3 y 4**. Calcula el tamaño restando: `5 - 2 = 3` letras.

## Transformar

```dart
void main() {
  final original = '  Hola, Mundo DAM  ';
  final limpio = original.trim();
  print('recortado: [$limpio]');
  print('mayúsculas: ${limpio.toUpperCase()}');
  print('minúsculas: ${limpio.toLowerCase()}');
  print('reemplazado: ${limpio.replaceAll('Mundo', 'Clase')}');

  final partes = limpio.split(', ');
  print('partes: ${partes.length} -> ${partes[0]} | ${partes[1]}');
  print('unido: ${partes.join('-')}');
  print('empieza por "Hola": ${limpio.startsWith('Hola') ? 'sí' : 'no'}');
  print('con ceros: ${'7'.padLeft(3, '0')}');
  print('original sigue igual: [$original]');
}
```

Salida:

```text
recortado: [Hola, Mundo DAM]
mayúsculas: HOLA, MUNDO DAM
minúsculas: hola, mundo dam
reemplazado: Hola, Clase DAM
partes: 2 -> Hola | Mundo DAM
unido: Hola-Mundo DAM
empieza por "Hola": sí
con ceros: 007
original sigue igual: [  Hola, Mundo DAM  ]
```

Observa la última línea: tras todos los cambios, **`original` no ha variado**, porque cada método devuelve una cadena nueva. Si quieres quedarte con el resultado, guárdalo en una variable (`limpio = original.strip()`), no basta con llamar al método.

## Los métodos más útiles

| Operación | En Dart |
|---|---|
| Longitud | `texto.length` |
| Un carácter | `texto[i]` |
| Subcadena (el final no se incluye) | `texto.substring(2, 5)` |
| Buscar posición (`-1` si no está) | `texto.indexOf('na')` |
| ¿Contiene? ¿Empieza? ¿Acaba? | `contains('nan')` · `startsWith('ba')` · `endsWith('na')` |
| Reemplazar | `replaceAll('a', 'o')` |
| Trocear / unir | `texto.split(', ')` · `partes.join('-')` |
| Quitar espacios de los extremos | `trim()` |
| Mayúsculas / minúsculas | `toUpperCase()` · `toLowerCase()` |
| Rellenar a la izquierda | `'7'.padLeft(3, '0')` |
| Repetir | `'ab' * 3` |

Para darle la vuelta rápidamente: `texto.split('').reversed.join()`. Y para construir un texto grande con muchos trozos, usa `StringBuffer` en vez de `+=` repetido.

## Unicode

`length` cuenta **unidades UTF-16**, no «letras» que ve el usuario: un emoji ocupa 2. Para texto con emojis o símbolos compuestos usa `texto.runes` o el paquete `characters`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a un método y esperar que cambie la cadena | Recoge el valor devuelto en una variable |
| Pasarse un puesto al recorrer (`<=` en lugar de `<`) | El último índice es la longitud menos 1 |
| Olvidar que el final de una subcadena no se incluye | Cuenta las letras: `final - inicio` |
| Comparar mayúsculas con minúsculas | Pasa los dos textos a minúsculas antes de comparar |

## Para practicar

Haz los [ejercicios 3.0 de cadenas](cadenas.md). Para ver cómo se escribe lo mismo en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
