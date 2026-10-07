# 2.3 Excepciones: cuando algo sale mal

Una **excepción** es un aviso de que ha ocurrido un error **mientras el programa se ejecuta** (convertir «abc» en número, dividir entre cero, abrir un archivo que no existe). Si nadie la controla, el programa **se detiene** y muestra un mensaje con la pila de llamadas.

Las excepciones permiten separar dos cosas: el camino normal del programa y qué hacer cuando algo falla.

## Capturar: `try`, `catch` y `finally`

```dart
void procesar(String texto) {
  try {
    final n = int.parse(texto);
    print('OK: $n -> doble ${n * 2}');
  } on FormatException {
    print('Error: «$texto» no es un número entero');
  } finally {
    print('-- procesado $texto');
  }
}

void main() {
  for (final texto in ['12', 'abc', '7']) {
    procesar(texto);
  }
}
```

Salida:

```text
OK: 12 -> doble 24
-- procesado 12
Error: «abc» no es un número entero
-- procesado abc
OK: 7 -> doble 14
-- procesado 7
```

* El bloque **`try`** contiene el código que puede fallar. Si todo va bien, se ejecuta entero.
* Si ocurre una excepción de ese tipo, el programa **salta** al bloque de captura y el resto del `try` no se ejecuta.
* El bloque **`finally`** se ejecuta **siempre**, haya o no excepción: es el sitio para cerrar archivos o liberar recursos.

En Dart, `on Tipo` captura un tipo concreto (`on FormatException`); `catch (e)` solo, captura **cualquiera**. Puedes combinar: `on FormatException catch (e)` para usar el objeto `e`. En Dart **no hay excepciones comprobadas**: nadie te obliga a capturarlas.

!!! tip "Captura el tipo concreto"
    Captura la excepción **más concreta posible**. Una captura genérica esconde errores que no esperabas y complica encontrarlos.

## Lanzar y crear excepciones

Tú también puedes avisar de un error: eso es **lanzar** una excepción. Se hace cuando un método recibe datos que no puede aceptar o cuando una operación no se puede completar.

```dart
class SaldoInsuficiente implements Exception {
  final int falta;
  SaldoInsuficiente(this.falta);
}

class Cuenta {
  int saldo;
  Cuenta(this.saldo);

  void retirar(int cantidad) {
    if (cantidad <= 0) {
      throw ArgumentError('la cantidad debe ser positiva');
    }
    if (cantidad > saldo) {
      throw SaldoInsuficiente(cantidad - saldo);
    }
    saldo -= cantidad;
  }
}

void main() {
  final cuenta = Cuenta(100);
  try {
    cuenta.retirar(30);
    print('Saldo: ${cuenta.saldo}');
    cuenta.retirar(200);
  } on SaldoInsuficiente catch (e) {
    print('Error: saldo insuficiente, faltan ${e.falta} €');
  }
  try {
    cuenta.retirar(-5);
  } on ArgumentError catch (e) {
    print('Error: ${e.message}');
  }
  print('Saldo final: ${cuenta.saldo}');
}
```

Salida:

```text
Saldo: 70
Error: saldo insuficiente, faltan 130 €
Error: la cantidad debe ser positiva
Saldo final: 70
```

Se lanza con **`throw`**. Dart ya trae `ArgumentError` (argumento incorrecto) y `FormatException`, y cualquier clase que implemente `Exception` sirve como excepción propia. En este ejemplo, `retirar` comprueba los datos antes de actuar y avisa con la excepción adecuada.

Fíjate en el orden: primero se comprueba que los datos son válidos y solo después se modifica el saldo. Así, si hay un error, el objeto **queda como estaba**.

## Un patrón útil: pedir un dato hasta que sea válido

Los ejercicios te piden a menudo que el programa no se rompa con lo que escriba el usuario. El patrón es un bucle que **repite la petición** hasta que el dato es correcto:

```dart
import 'dart:io';

void main() {
  while (true) {
    print('Escribe un número positivo:');
    final linea = stdin.readLineSync() ?? '';
    try {
      final n = int.parse(linea.trim());
      if (n <= 0) {
        throw FormatException('no es positivo');
      }
      print('Has escrito $n');
      break;
    } on FormatException {
      print('Eso no es un número positivo');
    }
  }
}
```

Si se escribe `abc`, luego `-3` y por último `42`:

```text
Escribe un número positivo:
Eso no es un número positivo
Escribe un número positivo:
Eso no es un número positivo
Escribe un número positivo:
Has escrito 42
```

Observa que un número negativo **no** es una excepción de la conversión, así que se lanza a mano para tratarlo igual que un texto no numérico.

## Cuándo usar una excepción

| Situación | Qué hacer |
|---|---|
| El usuario puede equivocarse al escribir | Captura la excepción y vuelve a pedir el dato |
| Un método recibe un argumento imposible | **Lánzala**: es un fallo de quien lo llama |
| Algo previsible (una lista vacía) | Compruébalo con un `if`: no abuses de las excepciones |
| Error que no sabes arreglar | Déjalo subir: que lo capture quien sepa qué hacer |

## Errores frecuentes

* **Capturar y no hacer nada.** Un bloque de captura vacío esconde el problema. Como mínimo, muestra o registra el error.
* **Capturar demasiado.** Pon en el `try` solo las líneas que pueden fallar.
* **Modificar el estado antes de validar**, de forma que un error deja el objeto a medias.

## Para practicar

Haz los [ejercicios 2.3 de excepciones](excepciones.md). El objetivo es que **ninguna excepción llegue al programa principal** y lo aborte.
