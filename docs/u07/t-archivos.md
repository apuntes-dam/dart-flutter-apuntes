# 7.B Archivos y carpetas

Los programas guardan datos en **archivos**, organizados en **carpetas** (directorios). Saber consultar, crear, copiar, mover y borrar es la base para cualquier programa que trabaje con datos que deben sobrevivir al cierre.

## Rutas

Una **ruta** indica dónde está un archivo o carpeta.

| Tipo | Ejemplo | Significa |
|---|---|---|
| **Absoluta** | `C:/Users/ana/datos/notas.txt` (Windows) · `/home/ana/datos/notas.txt` (Linux, macOS) | Desde la raíz del sistema |
| **Relativa** | `datos/notas.txt` | Desde la **carpeta de trabajo** del programa (la carpeta desde la que se ejecuta) |

!!! tip "Usa `/` en las rutas"
    Aunque Windows escribe las rutas con `\`, en Dart (y en los otros tres lenguajes) la barra normal `/` funciona en todos los sistemas y evita problemas con el carácter de escape `\`.

## Las operaciones básicas

Se usan las clases de **`dart:io`**: **`File`** (un archivo), **`Directory`** (una carpeta) y **`FileSystemEntity`**, que es la base de ambas y permite preguntar el tipo de una ruta (`FileSystemEntity.isDirectorySync(ruta)`). Casi todo tiene dos versiones: la **síncrona** (`...Sync`, que se usa aquí por sencillez) y la **asíncrona** (con `Future`), que no bloquea el programa. Para trabajar con rutas (unir, sacar el nombre, la extensión) existe el paquete `path` de pub.dev, que no es parte del SDK.

```dart
import 'dart:io';

String nombre(FileSystemEntity e) => e.uri.pathSegments.lastWhere((s) => s.isNotEmpty);

List<String> nombres(Directory carpeta) => carpeta.listSync().map(nombre).toList()..sort();

void main() {
  final carpeta = Directory('datos');
  Directory('datos/sub').createSync(recursive: true);
  print('carpeta creada: ${carpeta.existsSync() ? 'sí' : 'no'}');

  File('datos/notas.txt').writeAsStringSync('uno\ndos\n');
  for (final n in nombres(carpeta)) {
    final ruta = '${carpeta.path}/$n';
    if (FileSystemEntity.isDirectorySync(ruta)) {
      print('$n: carpeta');
    } else {
      print('$n: archivo de ${File(ruta).lengthSync()} bytes');
    }
  }

  File('datos/notas.txt').copySync('datos/notas_copia.txt');
  print('tras copiar: ${nombres(carpeta).join(', ')}');
  File('datos/notas_copia.txt').renameSync('datos/resumen.txt');
  print('tras renombrar: ${nombres(carpeta).join(', ')}');

  carpeta.deleteSync(recursive: true);
  print('datos existe tras borrar: ${carpeta.existsSync() ? 'sí' : 'no'}');
}
```

Salida:

```text
carpeta creada: sí
notas.txt: archivo de 8 bytes
sub: carpeta
tras copiar: notas.txt, notas_copia.txt, sub
tras renombrar: notas.txt, resumen.txt, sub
datos existe tras borrar: no
```

El programa crea una carpeta con una subcarpeta, escribe un archivo, **inspecciona** lo que hay (¿archivo o carpeta? ¿cuánto ocupa?), lo copia, lo renombra y lo borra todo al final. Fíjate en que la lista de nombres se **ordena** antes de mostrarla: el sistema no garantiza ningún orden al listar una carpeta.

| Necesito... | En Dart |
|---|---|
| ¿Existe? ¿Es carpeta? | `File(r).existsSync()` · `FileSystemEntity.isDirectorySync(r)` |
| Tamaño en bytes | `File(r).lengthSync()` |
| Crear carpeta (con las que falten) | `Directory(r).createSync(recursive: true)` |
| Listar el contenido | `Directory(r).listSync()` |
| Copiar · renombrar o mover | `File(r).copySync(destino)` · `File(r).renameSync(destino)` |
| Borrar archivo · carpeta con todo lo que contiene | `File(r).deleteSync()` · `Directory(r).deleteSync(recursive: true)` |

## Cuando algo falla

Un archivo que no existe o sin permisos lanza una **`FileSystemException`** (o una subclase, como `PathNotFoundException`).

Los fallos son **normales** con archivos (el usuario escribe mal una ruta, el disco está lleno, otro programa tiene el archivo abierto). Un programa robusto **comprueba antes** lo que pueda (¿existe?) y **captura** lo demás para dar un mensaje claro.

!!! danger "Cuidado con lo que borras"
    Borrar con `recursive`, `rmtree`, `deleteRecursively` o `walk` elimina **todo** lo que haya dentro, sin papelera y sin vuelta atrás. Antes de borrar:

    * Comprueba que la ruta es **la que esperas** (no vacía, no la raíz de un disco, no tu carpeta de usuario).
    * Si el borrado lo decide el usuario, **pídele confirmación**.
    * Mientras pruebas, usa una carpeta de prueba aparte.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Rutas relativas que dependen de dónde se ejecute el programa | Comprobar la carpeta de trabajo, o usar rutas absolutas |
| Asumir un orden al listar una carpeta | Ordenar los nombres |
| Sobrescribir un archivo existente sin avisar | Comprobar si existe y preguntar antes |
| Borrar una carpeta no vacía con la función de un solo archivo | Usar la versión recursiva, con confirmación |
| Olvidar cerrar lo que se abre (en Java, `Files.list`) | `try-with-resources` |

## Para practicar

Haz los ejercicios de [U7.2 · Archivos y carpetas](archivos.md). Y para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
