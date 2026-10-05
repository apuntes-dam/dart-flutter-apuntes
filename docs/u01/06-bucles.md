# 1.6 Bucles: `for` y `forEach`

Un bucle repite un bloque de código. En Dart hay dos formas muy usadas de recorrer valores: el `for` de toda la vida y el método `forEach`.

## Versión 1: `for` donde se ve de dónde sale cada cosa

```dart
void main() {
  for (var i = 1; i <= 5; i++) {
    print('Número: $i');
  }
}
```

Se lee de izquierda a derecha:

| Parte | Qué hace |
|---|---|
| `var i = 1` | **De dónde sale**: `i` nace valiendo 1 |
| `i <= 5` | **Hasta cuándo**: se repite mientras sea cierto |
| `i++` | **Cómo avanza**: suma 1 al terminar cada vuelta |
| `{ ... }` | Lo que se repite |

## Versión 2: sin `for` completo, con `forEach`

Dart no tiene rangos como `1..5`, así que se parte de una lista:

```dart
void main() {
  final numeros = [1, 2, 3, 4, 5];

  numeros.forEach((numero) {
    print('Número actual: $numero');
  });
}
```

`(numero) { ... }` es una **función anónima**: se ejecuta una vez por cada elemento y `numero` es el elemento de esa vuelta.

Si el cuerpo es una sola línea, se acorta con flecha:

```dart
numeros.forEach((numero) => print('Elemento: $numero'));
```

Con lógica añadida:

```dart
void main() {
  final numeros = [1, 2, 3, 4, 5];

  numeros.forEach((numero) {
    if (numero % 2 == 0) {
      print('$numero es par');
    } else {
      print('$numero es impar');
    }
  });
}
```

!!! note "No hay `it`"
    En Kotlin existe el parámetro implícito `it`. En Dart siempre se pone el nombre del parámetro.

## Comparación rápida

| Característica | `for` | `forEach` |
|---|---|---|
| Estilo | Tradicional | Funcional (función anónima) |
| `break` / `continue` | ✅ Sí | ❌ No (un `return` dentro solo salta a la siguiente vuelta) |
| `await` dentro | ✅ Sí | ❌ No espera de verdad |
| Legibilidad | Muy clara en bucles simples | Mejor con operaciones encadenadas |

!!! tip "Regla práctica"
    Empieza con `for`. Usa `forEach` cuando solo quieras hacer algo con cada elemento y no necesites cortar el bucle.

## Variantes

```dart
void main() {
  // Descendente (el equivalente de "downTo")
  for (var i = 5; i >= 1; i--) {
    print(i);
  }

  // Con paso de 2 (el equivalente de "step")
  for (var i = 1; i <= 10; i += 2) {
    print(i);
  }

  // Sin incluir el último valor (el equivalente de "until")
  for (var i = 1; i < 5; i++) {
    print(i);
  }

  // Recorrer una lista
  final nombres = ['Ana', 'Luis', 'Eva'];
  for (final nombre in nombres) {
    print(nombre);
  }

  // Recorrer con índice
  for (var i = 0; i < nombres.length; i++) {
    print('$i: ${nombres[i]}');
  }
}
```

## Cortar un bucle con una variable de control

Una forma sencilla de parar un bucle sin `break`: una variable booleana (una *bandera*) que el bucle consulta en su condición. Cuando pasa lo que buscas, la pones en `false`.

```dart
void main() {
  var activo = true;
  var i = 1;

  while (activo) {
    print(i);
    if (i == 5) {
      activo = false; // se cumple la condición: el bucle se detiene
    }
    i++;
  }
}
```

Con `do-while`, que se ejecuta al menos una vez:

```dart
void main() {
  var activo = true;
  var intentos = 0;

  do {
    intentos++;
    print('Intento $intentos');
    if (intentos == 3) {
      activo = false;
    }
  } while (activo);
}
```

!!! tip "Por qué esta forma"
    La condición del bucle muestra de un vistazo cuándo termina, y funciona igual en Dart, Java y Kotlin.

??? note "Opcional: `break` y `continue`"
    Existen, pero no son imprescindibles: casi siempre se puede escribir lo mismo con una variable de control.

    | Palabra | Qué hace |
    |---|---|
    | `break` | Sale del bucle de golpe |
    | `continue` | Salta a la siguiente vuelta |

    ```dart
    void main() {
      for (var i = 1; i <= 10; i++) {
        if (i == 3) continue; // salta esta vuelta
        if (i == 6) break;    // sale del bucle
        print(i);             // 1, 2, 4, 5
      }
    }
    ```

## `while` y `do-while`

```dart
void main() {
  var cuenta = 3;
  while (cuenta > 0) {   // comprueba antes
    print(cuenta--);
  }

  var intentos = 0;
  do {                   // se ejecuta al menos una vez
    intentos++;
  } while (intentos < 3);
  print(intentos);       // 3
}
```
