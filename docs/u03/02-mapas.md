# 3.2 Mapas (diccionarios)

Un **mapa** (o **diccionario**) guarda **pares clave → valor**. En lugar de buscar por posición, como en una lista, se busca por clave: dado un nombre, su edad; dada una palabra, cuántas veces aparece. Las **claves son únicas**: si asignas otra vez la misma clave, se sobrescribe el valor.

En Dart es `Map<K, V>`; el literal es `{'Ana': 25}` y conserva el **orden de inserción**.

## Crear, consultar, modificar y borrar

```dart
void main() {
  final edades = {'Ana': 25, 'Luis': 30};
  print('edad de Ana: ${edades['Ana']}');
  print('Pepe: ${edades['Pepe'] ?? 'no está'}');
  print('Pepe con valor por defecto: ${edades['Pepe'] ?? 0}');

  edades['Eva'] = 22; // añadir
  edades['Ana'] = 26; // modificar
  for (final entrada in edades.entries) {
    print('${entrada.key} -> ${entrada.value}');
  }
  edades.remove('Luis');
  print('tras borrar a Luis quedan ${edades.length}');

  final cuentas = <String, int>{};
  for (final palabra in 'a b a c b a'.split(' ')) {
    cuentas[palabra] = (cuentas[palabra] ?? 0) + 1;
  }
  print('frecuencias:');
  cuentas.forEach((clave, valor) => print('$clave -> $valor'));
}
```

Salida:

```text
edad de Ana: 25
Pepe: no está
Pepe con valor por defecto: 0
Ana -> 26
Luis -> 30
Eva -> 22
tras borrar a Luis quedan 2
frecuencias:
a -> 3
b -> 2
c -> 1
```

| Operación | En Dart |
|---|---|
| Consultar (`null` si no está) | `edades['Ana']` |
| Valor por defecto | `edades['Pepe'] ?? 0` |
| Añadir o modificar | `edades['Eva'] = 22` |
| ¿Existe la clave? | `edades.containsKey('Ana')` |
| Borrar | `edades.remove('Luis')` |
| Tamaño | `edades.length` |
| Solo claves · solo valores | `edades.keys` · `edades.values` |
| Recorrer pares | `for (final e in edades.entries)` → `e.key`, `e.value` |

!!! warning "Consultar una clave que no existe"
    Consultar una clave que no existe **no falla**: devuelve `null`. Por eso `edades['Ana']` tiene tipo `int?` y, si vas a operar con él, hay que decidir qué hacer con el `null` (`?? 0`, `!`, un `if`).

## Contar con un mapa

La segunda mitad del ejemplo es un patrón que se usa constantemente: **contar cuántas veces aparece cada elemento**. Para cada palabra, se lee su cuenta actual (o 0 si es la primera vez), se suma 1 y se guarda de nuevo.

## Claves, valores y mapas de listas

```dart
void main() {
  final notas = {
    'Ana': [7, 9],
    'Luis': [5, 6, 7],
  };
  print('claves: ${notas.keys.join(', ')}');
  for (final entrada in notas.entries) {
    final lista = entrada.value;
    var suma = 0;
    for (final n in lista) {
      suma += n;
    }
    print('${entrada.key}: media ${suma / lista.length}');
  }
  print('total de notas: ${notas.values.expand((lista) => lista).length}');
}
```

Salida:

```text
claves: Ana, Luis
Ana: media 8.0
Luis: media 6.0
total de notas: 5
```

Los valores pueden ser de cualquier tipo, incluidas **listas** u otros mapas. Aquí cada alumno tiene una lista de notas; se recorre el mapa y, para cada par, se recorre su lista.

## ¿Mapa, lista o conjunto?

| Necesito... | Estructura |
|---|---|
| Datos ordenados a los que accedo por posición | Lista |
| Buscar un dato a partir de otro (nombre → edad) | **Mapa** |
| Saber si algo está, sin repeticiones | Conjunto |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar el valor de una clave que no existe | Comprueba, o usa el valor por defecto |
| Esperar un orden concreto | Usa una versión ordenada si el orden importa |
| Modificar el mapa mientras lo recorres | Recorre una copia de las claves o construye un mapa nuevo |
| Repetir una clave pensando que añade otra entrada | Las claves son únicas: la segunda **sobrescribe** |

## Para practicar

Haz los [ejercicios 3.2 de mapas](mapas.md). Y para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
