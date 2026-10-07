# A3.A Tareas a la vez

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5 y la [unidad A1 de funciones como valores](../a1/index.md), porque las tareas se pasan como funciones.

Un programa **concurrente** tiene varias tareas **en marcha a la vez**. No siempre significa ejecutar a la vez de verdad: un solo procesador puede **alternar** entre tareas tan rápido que parece simultáneo. Cuando varias tareas se ejecutan **realmente** a la vez en varios núcleos, se habla de **paralelismo**.

| Tipo de tarea | Qué hace casi todo el tiempo | Ejemplos | Qué ayuda |
|---|---|---|---|
| **De espera** (*I/O-bound*) | **Esperar** a algo externo | Descargar, consultar una base de datos, leer un archivo | Alternar tareas (concurrencia) |
| **De cálculo** (*CPU-bound*) | **Calcular** sin parar | Comprimir, procesar una imagen, recorrer millones de datos | Repartir entre núcleos (paralelismo) |

## El modelo de Dart

Dart ejecuta tu código en un **solo hilo con un bucle de eventos**: una cola de tareas pendientes que va atendiendo una a una. Una función `async` **no crea un hilo nuevo**: cuando llega a un `await`, se aparta y deja que el bucle atienda otras cosas hasta que lo esperado esté listo. Para trabajo **pesado de cálculo** (que no espera, sino que consume procesador) se usa un **isolate**: otro hilo con **su propia memoria**, que no comparte variables con el principal y se comunica por mensajes. `Future` representa «un valor que llegará».

## Un ejemplo: tres tareas a la vez

Tres tareas simulan descargas de distinta duración (300, 100 y 200 ms). Se lanzan **todas a la vez**, se espera a que acaben y se comprueba el tiempo total:

```dart
// Tres tareas que "esperan" (como una descarga) y avanzan a la vez: el programa no se bloquea en cada una.
Future<void> tarea(String nombre, int milisegundos) async {
  await Future.delayed(Duration(milliseconds: milisegundos)); // se libera el hilo mientras espera
  print('termina $nombre');
}

Future<void> main() async {
  final reloj = Stopwatch()..start();
  final tareas = <Future<void>>[];
  for (final (nombre, ms) in [('A', 300), ('B', 100), ('C', 200)]) {
    print('empieza $nombre');
    tareas.add(tarea(nombre, ms)); // se lanza y todavía NO se espera
  }
  await Future.wait(tareas); // ahora sí: esperar a las tres
  print('las tres tareas juntas tardaron menos de 450 ms: ${reloj.elapsedMilliseconds < 450 ? "sí" : "no"}');
}
```

Salida:

```text
empieza A
empieza B
empieza C
termina B
termina C
termina A
las tres tareas juntas tardaron menos de 450 ms: sí
```

`Future.delayed` es una pausa que **no bloquea**: mientras la tarea espera, el bucle atiende a las demás.

Fíjate en dos cosas de la salida:

1. **Empiezan todas antes de que termine ninguna.** Se lanzan sin esperar.
2. **Terminan por duración, no por orden de lanzamiento**: B (100 ms), luego C (200 ms) y por último A (300 ms). El tiempo total es el de la **más lenta**, unos 300 ms, no la suma de las tres (600 ms).

!!! warning "Un orden que depende del tiempo"
    El orden en que terminan las tareas **no está garantizado por el lenguaje**: depende de cuánto tarden de verdad. En este ejemplo las pausas están muy separadas para que el resultado sea siempre el mismo. En un programa real, no escribas código que dependa de qué tarea termina antes.

## Qué usar en cada caso

| Si necesitas | En Dart |
|---|---|
| Esperar red, archivos o temporizadores | `async`/`await` y `Future` |
| Cálculo pesado sin congelar la interfaz | `Isolate.run` |
| Varias esperas a la vez | `Future.wait` |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Bloquear con una espera normal (`sleep`) dentro de una tarea asíncrona | Usa la espera propia del modelo (`delay`, `asyncio.sleep`, `Future.delayed`) |
| Lanzar una tarea y no esperarla | Guarda el `Future`/`Job`/tarea y espera a que termine |
| Dar por hecho el orden en que terminan | Si importa el orden, espera a cada una por separado |
| Crear un hilo por cada tarea pequeña | Usa un grupo de hilos o corrutinas |

## Para practicar

El ejercicio [A3.1](ejercicios.md) pide lanzar tres descargas a la vez, y el [A3.2](ejercicios.md) repartir una suma entre dos tareas. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
