# 1.1 Un programa

Un **programa** es una secuencia de instrucciones que un ordenador ejecuta para resolver un problema. Antes de escribirlo hay que tener claro el **algoritmo**: los pasos, en orden, que llevan de unos datos de entrada a un resultado.

## Ciclo de desarrollo

1. **Analizar** el problema: qué entra, qué debe salir.
2. **Diseñar** el algoritmo (pseudocódigo o diagrama).
3. **Codificar** en un lenguaje (aquí, Dart).
4. **Probar** y corregir.
5. **Documentar** y mantener.

## Pseudocódigo

El pseudocódigo describe el algoritmo sin la sintaxis de un lenguaje concreto.

```text
ALGORITMO areaRectangulo
  LEER base
  LEER altura
  area <- base * altura
  ESCRIBIR area
FIN
```

## Del pseudocódigo a Dart

```dart
import 'dart:io';

void main() {
  stdout.write('Base: ');
  final base = double.parse(stdin.readLineSync()!);
  stdout.write('Altura: ');
  final altura = double.parse(stdin.readLineSync()!);

  final area = base * altura;
  print('Área: $area');
}
```

| Pseudocódigo | Dart |
|---|---|
| `LEER x` | `stdin.readLineSync()` |
| `x <- expresión` | `x = expresión;` |
| `ESCRIBIR x` | `print(x)` |

!!! warning "Errores típicos al diseñar"
    Olvidar un caso (por ejemplo, base cero), no inicializar una variable antes de usarla y mezclar tipos sin convertir.
