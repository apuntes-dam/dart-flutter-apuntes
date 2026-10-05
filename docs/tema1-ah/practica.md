# Práctica · Hola, plataformas

Crea una app Flutter y ejecútala **sin cambiar el código** en al menos tres plataformas distintas.

## Pasos

1. Comprueba el entorno con `flutter doctor`.
2. Crea el proyecto:

    ```bash
    flutter create --org com.ejemplo hola_plataformas
    cd hola_plataformas
    ```

3. Mira qué dispositivos tienes:

    ```bash
    flutter devices
    ```

4. Sustituye `lib/main.dart` por tu versión de la app. Una posible solución, escrita por mí como referencia:

    ```dart
    import 'package:flutter/foundation.dart';
    import 'package:flutter/material.dart';

    void main() => runApp(const MiApp());

    String plataforma() {
      if (kIsWeb) return 'Web';
      return defaultTargetPlatform.name; // android, iOS, windows...
    }

    class MiApp extends StatelessWidget {
      const MiApp({super.key});

      @override
      Widget build(BuildContext context) {
        return MaterialApp(
          title: 'Hola, plataformas',
          theme: ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo)),
          home: const Inicio(),
        );
      }
    }

    class Inicio extends StatefulWidget {
      const Inicio({super.key});

      @override
      State<Inicio> createState() => _InicioState();
    }

    class _InicioState extends State<Inicio> {
      int _pulsaciones = 0;

      @override
      Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(title: const Text('Hola, plataformas')),
          body: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Text('¡Hola desde ${plataforma()}!',
                    style: Theme.of(context).textTheme.headlineMedium),
                const SizedBox(height: 8),
                Text('Has pulsado $_pulsaciones veces'),
              ],
            ),
          ),
          floatingActionButton: FloatingActionButton(
            onPressed: () => setState(() => _pulsaciones++),
            child: const Icon(Icons.add),
          ),
        );
      }
    }
    ```

5. Ejecútala en cada plataforma:

    ```bash
    flutter run -d chrome
    flutter run -d windows
    flutter run -d all        # todas a la vez
    ```

    Si te dice que falta la carpeta de una plataforma: `flutter create --platforms=web,windows .`

6. **Hot reload:** con la app en marcha, pulsa `+` hasta 5, cambia `Colors.indigo` por `Colors.teal` y pulsa `r`. El color cambia y el contador sigue en 5. `R` (mayúscula) reinicia y lo pierde.

## Entrega

* Capturas de la app en tres plataformas.
* La salida de `flutter devices`.
* Una frase explicando qué cambia entre plataformas y qué no.

!!! tip "Para distribuir"
    `flutter build web` genera una web en `build/web/`; `flutter build apk` genera el APK en `build/app/outputs/flutter-apk/`.
