# 7.D Interfaces gráficas

Hasta ahora los programas **leían** de la consola y **escribían** en ella, en un orden fijo. Una **interfaz gráfica** (GUI) cambia la idea: el programa muestra una **ventana** y **espera** a que el usuario haga algo (escribir, pulsar, elegir). Se dice que está **dirigido por eventos**.

## Ideas comunes a todas las librerías

| Idea | Qué es |
|---|---|
| **Componente** (*widget*) | Cada elemento de la ventana: texto, campo de entrada, botón, lista, imagen |
| **Contenedor y diseño** (*layout*) | Cómo se colocan los componentes: en columna, en fila, en rejilla |
| **Evento** | Algo que ocurre: una pulsación, una tecla, un clic |
| **Controlador de eventos** (*handler* o *callback*) | El código que se ejecuta cuando ocurre el evento |
| **Estado** | Los datos que cambian y que determinan lo que se ve (el texto escrito, el resultado) |
| **Bucle de eventos** | El ciclo interno que espera eventos y los reparte; se pone en marcha al abrir la ventana |

El programa no controla el orden: **reacciona**. Por eso cada botón lleva asociado su controlador.

En Dart la herramienta para crear interfaces es **Flutter**, un conjunto de **widgets** (componentes) que se combinan en un **árbol**: la pantalla es un `Scaffold` con una columna, que contiene un campo, un botón y un texto. Es un enfoque **declarativo**: describes **cómo debe verse la pantalla para un estado dado** y, cuando el estado cambia (`setState`), Flutter la vuelve a dibujar.

## Un ejemplo: la propina

La ventana tiene un campo para escribir un importe, un botón «Calcular» y un texto con el resultado (el 10 % del importe). Si lo escrito no es un número entero o es negativo, muestra un mensaje de error.

```dart
import 'package:flutter/material.dart';

String calcularPropina(String texto) {
  final importe = int.tryParse(texto.trim());
  if (importe == null) return 'Escribe un número entero';
  if (importe < 0) return 'El importe no puede ser negativo';
  return 'Propina: ${importe ~/ 10} €';
}

void main() => runApp(const PropinaApp());

class PropinaApp extends StatelessWidget {
  const PropinaApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(title: 'Propina', home: Pantalla());
  }
}

class Pantalla extends StatefulWidget {
  const Pantalla({super.key});

  @override
  State<Pantalla> createState() => _PantallaState();
}

class _PantallaState extends State<Pantalla> {
  final _controlador = TextEditingController();
  String _resultado = '';

  @override
  void dispose() {
    _controlador.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Propina')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _controlador,
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(labelText: 'Importe (€)'),
            ),
            const SizedBox(height: 12),
            FilledButton(
              onPressed: () => setState(() => _resultado = calcularPropina(_controlador.text)),
              child: const Text('Calcular'),
            ),
            const SizedBox(height: 12),
            Text(_resultado, style: Theme.of(context).textTheme.titleMedium),
          ],
        ),
      ),
    );
  }
}
```

Crea un proyecto con `flutter create propina`, sustituye el contenido de `lib/main.dart` por este código y ejecútalo con `flutter run -d windows` (o `-d chrome`).

Hay una decisión de diseño importante: **`calcularPropina` es una función aparte**, que recibe un texto y devuelve otro, sin saber nada de ventanas. La pantalla solo se ocupa de **recoger** el texto, **llamar** a la función y **mostrar** el resultado. Esa separación entre **lógica** e **interfaz** hace el código más fácil de probar, de reutilizar (la misma función serviría en consola o en una web) y de cambiar.

## Probar la ventana sin abrirla

Flutter permite hacer **pruebas de widgets**: se construye la pantalla **sin abrirla**, se escribe en el campo, se pulsa el botón y se comprueba lo que muestra. Se ejecutan con `flutter test`.

```dart
// Prueba de widgets: escribe en el campo, pulsa el botón y busca el texto resultante.
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:ejemplo/main.dart';

void main() {
  for (final (entrada, esperado) in [
    ('50', 'Propina: 5 €'),
    ('abc', 'Escribe un número entero'),
    ('-5', 'El importe no puede ser negativo'),
  ]) {
    testWidgets('$entrada -> $esperado', (tester) async {
      await tester.pumpWidget(const PropinaApp());
      await tester.enterText(find.byType(TextField), entrada);
      await tester.tap(find.text('Calcular'));
      await tester.pump();
      expect(find.text(esperado), findsOneWidget);
    });
  }
}
```

Cambia `ejemplo` por el nombre de tu proyecto en el `import`.

Resultado de los tres casos:

```text
50 -> Propina: 5 €
abc -> Escribe un número entero
-5 -> El importe no puede ser negativo
```

He ejecutado estas pruebas de widgets con Flutter 3.47: las tres pasan.

## Tres cuidados importantes

* **Nunca bloquees la ventana.** Mientras el controlador de un evento está trabajando, la ventana **no responde**. Las tareas largas (descargas, cálculos pesados, lecturas de archivos grandes) deben hacerse **fuera del hilo de la interfaz**.
* **Valida lo que escribe el usuario.** Todo lo que llega de un campo es **texto**: conviértelo y prevé que falle, como hace `calcularPropina`.
* **Cambia la interfaz desde el sitio correcto.** En Flutter, el estado se cambia dentro de `setState`; fuera de él, la pantalla no se actualiza.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| La pantalla no se actualiza al cambiar un dato | Cambiar el estado **del modo que la librería exige** (ver arriba) |
| Escribir la lógica dentro del controlador del botón | Ponerla en una función aparte |
| Congelar la ventana con una tarea larga | Hacerla fuera del hilo de la interfaz |
| Fiarse de que el usuario escribirá un número | Validar y mostrar un mensaje claro |
| Componentes que se salen o se superponen | Usar un contenedor con diseño (columna, rejilla) en vez de posiciones fijas |

## Para practicar

Haz los ejercicios de [U7.4 · Interfaces gráficas](gui.md): una ventana con botón, un contador con estado, un formulario con validación y una lista de tareas. Si quieres profundizar en Compose, tienes la [web de Android y apps móviles](https://apuntes-dam.github.io/android-apuntes/), que trata botones, listas, formularios y navegación. [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) compara el código de cada lenguaje.
