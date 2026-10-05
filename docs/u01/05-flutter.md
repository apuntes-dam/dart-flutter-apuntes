# 1.5 Flutter: primera app

**Flutter** dibuja la interfaz con **widgets**: todo es un widget (texto, botón, columna, la app entera). Los widgets se anidan formando un árbol.

## Crear y ejecutar

```bash
flutter create mi_app
cd mi_app
flutter run
```

## Widgets sin estado y con estado

| Tipo | Cuándo | Ejemplo |
|---|---|---|
| `StatelessWidget` | La pantalla no cambia con el tiempo | Un título, un icono |
| `StatefulWidget` | Hay datos que cambian | Un contador, un formulario |

## App mínima

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MiApp());

class MiApp extends StatelessWidget {
  const MiApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: Scaffold(
        body: Center(child: Text('Hola, Flutter')),
      ),
    );
  }
}
```

## Contador con estado

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: Contador()));

class Contador extends StatefulWidget {
  const Contador({super.key});

  @override
  State<Contador> createState() => _ContadorState();
}

class _ContadorState extends State<Contador> {
  int _cuenta = 0;

  void _incrementar() {
    setState(() => _cuenta++);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(child: Text('Pulsaciones: $_cuenta')),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementar,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

!!! note "`setState`"
    Avisa a Flutter de que el estado cambió para que vuelva a ejecutar `build`. Modificar `_cuenta` sin `setState` no actualiza la pantalla.

## Hot reload

Con la app en marcha, pulsar `r` en la terminal (o guardar en el IDE) recarga el código manteniendo el estado. `R` reinicia desde cero.
