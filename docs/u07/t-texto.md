# 7.C Ficheros de texto

Un **fichero de texto** guarda caracteres legibles (notas, configuraciones, registros, CSV, JSON...). Se puede abrir con cualquier editor, a diferencia de un fichero **binario** (una imagen, un ejecutable). Leer y escribir texto es lo más habitual al trabajar con datos.

## Crear, añadir y leer

`File.writeAsStringSync(texto)` **crea o sobrescribe** el archivo; para **añadir** al final se pasa `mode: FileMode.append`. `readAsLinesSync()` devuelve la lista de líneas y `readAsStringSync()`, todo el contenido de golpe. Estos métodos abren y **cierran** el archivo por ti. Para ficheros muy grandes existe `openRead()`, que lee por trozos (de forma asíncrona) sin cargar todo en memoria. La codificación por defecto es UTF-8.

```dart
import 'dart:io';

void main() {
  final archivo = File('texto.txt');
  archivo.writeAsStringSync('primera línea\nsegunda línea\n'); // crea o sobrescribe
  archivo.writeAsStringSync('tercera\n', mode: FileMode.append); // añade al final

  final lineas = archivo.readAsLinesSync();
  print('${lineas.length} líneas');
  var palabras = 0;
  for (var i = 0; i < lineas.length; i++) {
    print('${i + 1}: ${lineas[i]}');
    palabras += lineas[i].split(' ').length;
  }
  print('palabras: $palabras');

  try {
    File('falta.txt').readAsStringSync();
  } on FileSystemException {
    print("No se pudo abrir 'falta.txt': el archivo no existe");
  }

  final copia = File('copia.txt');
  copia.writeAsStringSync(lineas.map((l) => l.toUpperCase()).join('\n') + '\n');
  print('primera línea en mayúsculas: ${copia.readAsLinesSync().first}');

  archivo.deleteSync();
  copia.deleteSync();
}
```

Salida:

```text
3 líneas
1: primera línea
2: segunda línea
3: tercera
palabras: 5
No se pudo abrir 'falta.txt': el archivo no existe
primera línea en mayúsculas: PRIMERA LÍNEA
```

Qué hace el programa, paso a paso:

1. **Escribe** dos líneas y después **añade** una tercera (sin borrar las anteriores).
2. **Lee** el archivo completo como lista de líneas y las muestra numeradas, contando las palabras.
3. **Intenta abrir un archivo que no existe** y avisa con un mensaje claro en lugar de dejar que el programa se detenga. Abrir un archivo que no existe lanza una **`FileSystemException`**.
4. **Copia** el archivo en mayúsculas leyendo **línea a línea**, y comprueba el resultado.
5. **Borra** los archivos de prueba.

## Tres decisiones al abrir un archivo

| Decisión | Opciones |
|---|---|
| **¿Qué hago con lo que ya hay?** | **Sobrescribir** (el contenido anterior se pierde) o **añadir** al final |
| **¿Cómo lo leo?** | **Entero** de una vez (cómodo, solo para ficheros pequeños) o **línea a línea** (cualquier tamaño, gasta poca memoria) |
| **¿Con qué codificación?** | **UTF-8**, que sirve para tildes, eñes y símbolos. Si el archivo se guardó con otra, las tildes salen mal |

!!! warning "Escribir borra"
    La operación de «escribir» **vacía** el archivo si ya existía. Si no quieres perder su contenido, **añade** en lugar de escribir, o pregunta antes de sobrescribir (como piden los ejercicios).

## Cerrar siempre el archivo

Un archivo abierto ocupa recursos del sistema y puede quedar **a medias** (con datos aún sin escribir en el disco) si no se cierra. Por eso se abre dentro de una estructura que lo **cierra automáticamente**, incluso si hay un error en medio. En Dart: las funciones de una sola llamada ya lo cierran; para el trabajo por líneas existe `openRead`/`openWrite` y hay que cerrarlos al terminar.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Perder el contenido anterior al escribir | Añadir (`append`) o preguntar antes de sobrescribir |
| Cargar en memoria un archivo enorme | Leer línea a línea |
| Tildes que se ven mal (`Ã¡` en vez de `á`) | Indicar UTF-8 al abrir (en Python, siempre `encoding`) |
| No cerrar el archivo | `with` / `use` / `try-with-resources` |
| Dar por hecho que el archivo existe | Capturar el error o comprobar antes |
| Contar las líneas sin tener en cuenta la última línea vacía | Probar con archivos que terminen y no terminen en salto de línea |

## Para practicar

Haz los ejercicios de [U7.3 · Ficheros de texto](texto.md). [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) muestra cómo se escribe lo mismo en otro lenguaje.
