# 3.4 JSON

**JSON** (*JavaScript Object Notation*) es un formato de **texto** para guardar e intercambiar datos. Es el formato más usado entre aplicaciones y servidores web, porque es compacto, fácil de leer para una persona y todos los lenguajes saben procesarlo.

```json
{
  "usuarios": [
    {"id": 1, "nombre": "Juan", "edad": 30},
    {"id": 2, "nombre": "Ana", "edad": 25}
  ]
}
```

## Qué contiene un JSON

| En JSON | Ejemplo | En Dart es... |
|---|---|---|
| Objeto `{ }` | `{"id": 1}` | un mapa (o un objeto de una clase) |
| Array `[ ]` | `[1, 2, 3]` | una lista |
| Texto | `"Ana"` | una cadena |
| Número | `25` · `3.5` | un entero o decimal |
| Booleano | `true` · `false` | un booleano |
| Nulo | `null` | el valor nulo |

Las reglas son estrictas: las claves y los textos van **siempre entre comillas dobles**, **no se admite una coma al final** de una lista ni de un objeto y **no hay comentarios**. La mayoría de errores con JSON son una de estas tres cosas.

Dart incluye la librería **`dart:convert`**: `jsonDecode(texto)` convierte el texto en mapas y listas, y `jsonEncode(datos)` hace lo contrario. Para escribir el resultado con sangría se usa `JsonEncoder.withIndent('  ')`. Los valores llegan como `dynamic`, así que a menudo hay que indicar el tipo (`as Map<String, dynamic>`).

## Leer, modificar y escribir

```dart
import 'dart:convert';

void mostrar(Map<String, dynamic> datos) {
  for (final u in datos['usuarios']) {
    print('ID: ${u['id']}, Nombre: ${u['nombre']}, Edad: ${u['edad']}');
  }
}

void main() {
  const texto =
      '{"usuarios": [{"id": 1, "nombre": "Juan", "edad": 30}, {"id": 2, "nombre": "Ana", "edad": 25}]}';
  final datos = jsonDecode(texto) as Map<String, dynamic>;
  mostrar(datos);

  final usuarios = datos['usuarios'] as List;
  usuarios[1]['edad'] = 26; // actualizar
  usuarios.add({'id': 3, 'nombre': 'Eva', 'edad': 22}); // insertar
  usuarios.removeWhere((u) => u['id'] == 1); // eliminar

  print('--- después de los cambios ---');
  mostrar(datos);
  print(JsonEncoder.withIndent('  ').convert(datos));
}
```

Salida:

```text
ID: 1, Nombre: Juan, Edad: 30
ID: 2, Nombre: Ana, Edad: 25
--- después de los cambios ---
ID: 2, Nombre: Ana, Edad: 26
ID: 3, Nombre: Eva, Edad: 22
{
  "usuarios": [
    {
      "id": 2,
      "nombre": "Ana",
      "edad": 26
    },
    {
      "id": 3,
      "nombre": "Eva",
      "edad": 22
    }
  ]
}
```

El proceso siempre es el mismo en tres pasos:

1. **Convertir el texto** en estructuras del lenguaje (mapas, listas u objetos).
2. **Trabajar con ellas** como con cualquier mapa o lista. Se actualiza un valor asignándolo (`usuarios[1]['edad'] = 26`), se inserta con `add` y se elimina con `removeWhere`, como con cualquier lista o mapa.
3. **Convertirlas de nuevo en texto**, con sangría si lo van a leer personas.

## Trabajar con archivos y controlar los errores

Los datos suelen estar en un **archivo**. Al leerlo pueden pasar dos cosas que hay que controlar siempre: que el archivo **no exista** o que su contenido **no sea un JSON válido**. Un JSON mal formado lanza una **`FormatException`**.

```dart
import 'dart:convert';
import 'dart:io';

List<dynamic>? cargar(String ruta) {
  final archivo = File(ruta);
  if (!archivo.existsSync()) {
    print("No existe el archivo '$ruta'");
    return null;
  }
  try {
    final datos = jsonDecode(archivo.readAsStringSync());
    return datos['usuarios'] as List<dynamic>;
  } on FormatException {
    print("El archivo '$ruta' no contiene un JSON válido");
    return null;
  }
}

void main() {
  cargar('no_existe.json');

  File('malo.json').writeAsStringSync('{ esto no es json');
  cargar('malo.json');

  final datos = {
    'usuarios': [
      {'id': 1, 'nombre': 'Juan', 'edad': 30},
      {'id': 2, 'nombre': 'Ana', 'edad': 25},
    ],
  };
  File('bueno.json').writeAsStringSync(jsonEncode(datos));
  final usuarios = cargar('bueno.json');
  print('Cargados ${usuarios?.length} usuarios');

  File('malo.json').deleteSync();
  File('bueno.json').deleteSync();
}
```

Salida:

```text
No existe el archivo 'no_existe.json'
El archivo 'malo.json' no contiene un JSON válido
Cargados 2 usuarios
```

Comprobar la existencia **antes** de abrir y capturar el error de formato permite dar un mensaje claro en vez de que el programa se detenga. Los archivos de texto se guardan en **UTF-8**, la codificación habitual del JSON.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Comillas simples en el texto del JSON | En JSON solo valen las **dobles** |
| Una coma de más al final | Quítala: JSON no la permite |
| Esperar un número y recibir texto (o al revés) | Comprueba el tipo o convierte |
| Pedir una clave que no existe | Comprueba con las consultas seguras del apartado 3.2 |
| Perder las tildes al guardar | Usa UTF-8 en lectura y escritura |

## Para practicar

Haz el [ejercicio 3.4 de JSON](json.md), la gestión de usuarios en un archivo.
