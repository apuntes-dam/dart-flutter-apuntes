# 4.B Encapsulamiento

**Encapsular** es **ocultar el estado interno** de un objeto y dejar solo unas operaciones controladas para usarlo. Así el objeto se encarga de que sus datos siempre tengan sentido: un precio nunca es negativo, un stock nunca baja de cero.

Sin encapsulamiento, cualquiera podría escribir `producto.precio = -5` y el error aparecería mucho más tarde, lejos de donde se produjo. Con él, el fallo salta **justo al intentar el cambio incorrecto**.

## Visibilidad

Dart no tiene `private` ni `public`: un nombre que **empieza por guion bajo** (`_precio`) es privado **para la biblioteca** (el archivo), no solo para la clase. Todo lo demás es público.

## Acceso controlado y validación

```dart
class Producto {
  final String nombre;
  int _precio;
  int _stock;

  Producto(this.nombre, int precio, int stock)
      : _precio = precio,
        _stock = stock {
    _comprobarPrecio(precio);
  }

  int get precio => _precio;
  int get stock => _stock;
  int get valorStock => _precio * _stock;

  set precio(int nuevo) {
    _comprobarPrecio(nuevo);
    _precio = nuevo;
  }

  void vender(int cantidad) {
    if (cantidad > _stock) {
      throw StateError('no hay stock suficiente');
    }
    _stock -= cantidad;
  }

  static void _comprobarPrecio(int precio) {
    if (precio < 0) {
      throw ArgumentError('el precio no puede ser negativo');
    }
  }
}

void main() {
  final p = Producto('Cuaderno', 3, 10);
  print('${p.nombre}: ${p.precio} € x ${p.stock} = ${p.valorStock} €');
  p.vender(4);
  print('tras vender 4, quedan ${p.stock}');
  try {
    p.vender(20);
  } on StateError catch (e) {
    print('Error: ${e.message}');
  }
  try {
    p.precio = -1;
  } on ArgumentError catch (e) {
    print('Error: ${e.message}');
  }
  p.precio = 7;
  print('nuevo valor del stock: ${p.valorStock} €');
  try {
    Producto('Roto', -2, 1);
  } on ArgumentError catch (e) {
    print('Error: ${e.message}');
  }
}
```

Salida:

```text
Cuaderno: 3 € x 10 = 30 €
tras vender 4, quedan 6
Error: no hay stock suficiente
Error: el precio no puede ser negativo
nuevo valor del stock: 42 €
Error: el precio no puede ser negativo
```

Qué hace cada pieza:

* **Atributos protegidos** (`precio`, `stock`): no se pueden cambiar desde fuera sin pasar por el código de la clase.
* **Validar al crear y al modificar**: el constructor y el cambio de precio comprueban el valor y **lanzan una excepción** si no es válido (ver [2.3 Excepciones](../u02/02-excepciones.md)).
* **Solo lectura**: el stock se puede consultar, pero **solo** `vender` lo modifica.
* **Propiedad calculada**: `valorStock` no se guarda; se calcula cada vez a partir de `precio` y `stock`, así nunca queda desactualizada.

Se usan **`get`** y **`set`**, que desde fuera parecen un atributo normal: `p.precio` y `p.precio = 7` llaman a código con validación. Una propiedad calculada, como `valorStock`, es un `get` sin atributo detrás.

!!! tip "Valida antes de modificar"
    Fíjate en `vender`: primero comprueba que hay stock y **solo después** resta. Si la comprobación falla, el objeto queda como estaba. Modificar primero y validar después deja objetos a medias.

## Igualdad: identidad frente a valor

Dos objetos pueden ser **el mismo** (la misma referencia) o ser **iguales** (tener el mismo contenido). Por defecto, `==` solo reconoce lo primero; para que dos puntos con las mismas coordenadas sean iguales hay que decírselo.

```dart
class PuntoSimple {
  final int x, y;
  PuntoSimple(this.x, this.y);
}

class Punto {
  final int x, y;
  Punto(this.x, this.y);

  @override
  bool operator ==(Object otro) => otro is Punto && x == otro.x && y == otro.y;

  @override
  int get hashCode => Object.hash(x, y);
}

void main() {
  print('sin igualdad por valor: ${PuntoSimple(1, 2) == PuntoSimple(1, 2) ? 'sí' : 'no'}');
  print('con igualdad por valor: ${Punto(1, 2) == Punto(1, 2) ? 'sí' : 'no'}');
  final conjunto = {Punto(1, 2), Punto(1, 2), Punto(3, 4)};
  print('puntos distintos en el conjunto: ${conjunto.length}');
}
```

Salida:

```text
sin igualdad por valor: no
con igualdad por valor: sí
puntos distintos en el conjunto: 2
```

En Dart hay que **sobrescribir `==` y `hashCode`** a la vez (el ejemplo usa `Object.hash`). Si no se hace, dos objetos son iguales solo si son el mismo. El paquete `equatable` lo simplifica.

Por la misma razón, en un **conjunto** o como **clave de un mapa**, la igualdad por valor decide si dos objetos son «el mismo elemento» (ver [3.3 Conjuntos](../u03/03-conjuntos.md)).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Atributos públicos y modificables desde cualquier sitio | Hazlos privados y ofrece métodos para lo que haga falta |
| Un *setter* que acepta cualquier valor | Valida y lanza una excepción si no es correcto |
| Ofrecer un *setter* para todo «por si acaso» | Solo lo que de verdad deba poder cambiar: lo demás, de solo lectura |
| Validar en el constructor pero no en el *setter* (o al revés) | El mismo control en **todas** las puertas de entrada |
| Sobrescribir `equals` sin `hashCode` (o al revés) | Siempre juntos y con los mismos campos |

## Para practicar

Los ejercicios de validación y atributos de solo lectura están en [U4.2 · POO I](poo-1.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
