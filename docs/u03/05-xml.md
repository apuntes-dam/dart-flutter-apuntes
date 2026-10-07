# 3.5 XML

**XML** (*eXtensible Markup Language*) es otro formato de texto para guardar datos **con estructura de árbol**. Es más antiguo y más verboso que JSON, pero sigue muy presente en configuraciones, documentos, servicios web antiguos y en el propio Android.

```xml
<usuarios>
  <usuario>
    <id>1</id>
    <nombre>Juan</nombre>
    <edad>30</edad>
  </usuario>
</usuarios>
```

## Partes de un XML

| Parte | Qué es | Ejemplo |
|---|---|---|
| **Elemento** | Una etiqueta de apertura, su contenido y la de cierre | `<nombre>Juan</nombre>` |
| **Atributo** | Un dato dentro de la etiqueta de apertura | `<usuario id="1">` |
| **Texto** | El contenido de un elemento | `Juan` |
| **Raíz** | El elemento que contiene a todos los demás (solo hay **uno**) | `<usuarios>` |

Para que un XML sea **válido** (*bien formado*): hay una sola raíz, cada etiqueta que se abre **se cierra** en el orden correcto, los atributos van entre comillas y se distingue entre mayúsculas y minúsculas. Los caracteres especiales se escriben con entidades: `&lt;` (`<`), `&gt;` (`>`) y `&amp;` (`&`).

El SDK de Dart **no incluye** un lector de XML: se usa el paquete **`xml`** de pub.dev. Se añade con `dart pub add xml` y se importa con `import 'package:xml/xml.dart';`. Estos ejemplos se han probado con la serie 6 del paquete.

## Leer, modificar y escribir

```dart
import 'package:xml/xml.dart';

String texto(XmlElement padre, String etiqueta) => padre.getElement(etiqueta)!.innerText;

void mostrar(XmlElement raiz) {
  for (final u in raiz.findElements('usuario')) {
    print('ID: ${texto(u, 'id')}, Nombre: ${texto(u, 'nombre')}, Edad: ${texto(u, 'edad')}');
  }
}

XmlElement elemento(String nombre, String contenido) =>
    XmlElement(XmlName(nombre), [], [XmlText(contenido)]);

void main() {
  const xml = '<usuarios><usuario><id>1</id><nombre>Juan</nombre><edad>30</edad></usuario>'
      '<usuario><id>2</id><nombre>Ana</nombre><edad>25</edad></usuario></usuarios>';
  final raiz = XmlDocument.parse(xml).rootElement;
  mostrar(raiz);

  for (final u in raiz.findElements('usuario')) {
    if (texto(u, 'nombre') == 'Ana') {
      u.getElement('edad')!.innerText = '26'; // actualizar
    }
  }
  raiz.children.add(XmlElement(XmlName('usuario'), [], [
    elemento('id', '3'),
    elemento('nombre', 'Eva'),
    elemento('edad', '22'),
  ])); // insertar
  raiz.findElements('usuario').firstWhere((u) => texto(u, 'id') == '1').remove(); // eliminar

  print('--- después de los cambios ---');
  mostrar(raiz);
  print(raiz.toXmlString());
}
```

Salida:

```text
ID: 1, Nombre: Juan, Edad: 30
ID: 2, Nombre: Ana, Edad: 25
--- después de los cambios ---
ID: 2, Nombre: Ana, Edad: 26
ID: 3, Nombre: Eva, Edad: 22
<usuarios><usuario><id>2</id><nombre>Ana</nombre><edad>26</edad></usuario><usuario><id>3</id><nombre>Eva</nombre><edad>22</edad></usuario></usuarios>
```

La idea es la misma que con JSON: cargar el texto como un **árbol**, buscar los elementos que interesan (`usuario`, y dentro `id`, `nombre`, `edad`), modificar el árbol y volver a convertirlo en texto. El texto de entrada de este ejemplo está en **una sola línea** para que la salida sea idéntica en los cuatro lenguajes; en un archivo real lo normal es escribirlo con sangría.

## Crear un árbol, atributos y errores

```dart
import 'package:xml/xml.dart';

void main() {
  final raiz = XmlElement(XmlName('usuarios'), [], [
    XmlElement(XmlName('usuario'), [XmlAttribute(XmlName('id'), '1')], [XmlText('Juan')]),
  ]);
  print(raiz.toXmlString());
  print('atributo id: ${raiz.getElement('usuario')!.getAttribute('id')}');

  try {
    XmlDocument.parse('<usuarios><usuario></usuarios>');
  } on XmlException {
    print('El texto no es un XML válido');
  }
}
```

Salida:

```text
<usuarios><usuario id="1">Juan</usuario></usuarios>
atributo id: 1
El texto no es un XML válido
```

Este ejemplo muestra tres cosas: **crear** un árbol desde cero (un XML vacío con su raíz es el punto de partida cuando un archivo no existe), **leer un atributo** (`id`) y **detectar un XML inválido**. Un XML mal formado lanza una **`XmlException`** (concretamente una `XmlParserException` o una `XmlTagException`).

## JSON o XML

| | JSON | XML |
|---|---|---|
| Aspecto | Compacto | Más verboso (etiquetas de apertura y cierre) |
| Estructura | Objetos y arrays | Árbol de elementos, con atributos |
| Comentarios | No | Sí |
| Uso típico | APIs web, configuración | Documentos, configuraciones, Android |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Olvidar cerrar una etiqueta o cerrarla en otro orden | Comprueba que el XML sea válido antes de procesarlo |
| Más de un elemento raíz | Un documento tiene **una** sola raíz |
| `&` o `<` sueltos dentro del texto | Escríbelos como `&amp;` y `&lt;` |
| Asumir que un elemento existe | Comprueba que el resultado de la búsqueda no sea nulo |

## Para practicar

Haz el [ejercicio 3.5 de XML](xml.md), igual que el de JSON pero con un árbol de elementos.
