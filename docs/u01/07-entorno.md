# 1.7 Entorno: instalar y comprobar

Flutter funciona con una idea sencilla: **un único SDK** (Flutter, que ya trae Dart dentro) y, por cada plataforma de destino, las herramientas nativas de esa plataforma.

## Qué hace falta

| Pieza | Para qué | ¿Obligatoria? |
|---|---|---|
| Git | Descargar Flutter y versionar | Sí |
| Flutter SDK | Framework y comandos `flutter` y `dart` | Sí |
| Editor con plugins Flutter y Dart | Escribir, depurar, *hot reload* | Sí |
| Android Studio | Android SDK y emulador | Para Android |
| Chrome o Edge | Ejecutar en web | Recomendado |
| Visual Studio (C++ de escritorio) | Escritorio Windows | Opcional |
| Xcode | iOS y macOS | Solo en Mac |

!!! warning "iOS solo desde Mac"
    Apple solo permite compilar para iOS y macOS desde un Mac. Desde Windows o Linux se puede compilar para Android, web y el escritorio del propio sistema.

## Instalación en Windows (resumen)

```bash
git --version
mkdir C:\develop
cd C:\develop
git clone https://github.com/flutter/flutter.git -b stable
```

Después, añade `C:\develop\flutter\bin` al `PATH`, abre una terminal nueva y comprueba:

```bash
flutter --version
dart --version
```

!!! tip "Ruta sin espacios ni tildes"
    Instala el SDK en una ruta como `C:\develop`, no dentro de la carpeta de usuario si tiene espacios o tildes.

## `flutter doctor`

```bash
flutter doctor
```

Revisa el entorno y dice cómo arreglar cada problema.

| Marca | Significado |
|---|---|
| `[✓]` | Correcto |
| `[!]` | Funciona a medias: hay algo que corregir |
| `[✗]` | No instalado o no funciona |

No hace falta ver todo en verde, solo lo de las plataformas para las que vayas a compilar. Para más detalle: `flutter doctor -v`.

## Problemas frecuentes

??? failure "`flutter` no se reconoce como un comando"
    La carpeta `flutter\bin` no está en el `PATH`. Añádela y **abre una terminal nueva**.

??? failure "Unable to find Android Studio"
    Indica a Flutter dónde está instalado:
    ```bash
    flutter config --android-studio-dir "RUTA_DE_ANDROID_STUDIO"
    ```

??? failure "Waiting for another flutter command to release the startup lock"
    Hay otro comando de Flutter en marcha o que se cortó. Espera, cierra las terminales abiertas y vuelve a probar.

??? failure "El emulador no aparece en `flutter devices`"
    Arráncalo desde el Device Manager de Android Studio y vuelve a ejecutar `flutter devices`. Para un móvil real, activa la depuración USB.

??? failure "Licencias de Android sin aceptar"
    ```bash
    flutter doctor --android-licenses
    ```

## De Java o Kotlin a Dart

| Idea | Java | Kotlin | Dart |
|---|---|---|---|
| Entero | `int`, `long` | `Int`, `Long` | `int` |
| Booleano | `boolean` | `Boolean` | `bool` |
| Constante | `final` | `val` | `final` / `const` |
| División entera | `7 / 2` con `int` | `7 / 2` con `Int` | `7 ~/ 2` |
| Plantilla de texto | `"a" + b` | `"a $b"` | `'a $b'` |
| Anulable | no existe | `String?` | `String?` |
| Recorrer | `for (String p : lista)` | `for (p in lista)` | `for (final p in lista)` |
