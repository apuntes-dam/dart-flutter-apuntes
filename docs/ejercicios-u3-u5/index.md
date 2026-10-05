# Ejercicios de Dart · Unidades 3 a 5

Continuación de los [ejercicios de Programación](../ejercicios/index.md): **65 ejercicios** de cadenas, colecciones, JSON/XML y programación orientada a objetos, adaptados a Dart. Los enunciados están redactados de nuevo a partir de los de las unidades 3, 4 y 5 de Programación.

| Bloque | Ejercicios |
|---|---|
| [U3.0 · Cadenas](u3-0.md) | 4 |
| [U3.1 · Listas y tuplas](u3-1.md) | 13 |
| [U3.2 · Mapas (diccionarios)](u3-2.md) | 11 |
| [U3.3 · Conjuntos](u3-3.md) | 6 |
| [U3.4 · JSON](u3-4.md) | 1 |
| [U3.5 · XML](u3-5.md) | 1 |
| [U4.1 · Repaso de las unidades 1 a 3](u4-1.md) | 1 |
| [U4.2 · POO I (ejercicios 1 al 5)](u4-2.md) | 5 |
| [U4.3 · POO II (ejercicios 6 al 10)](u4-3.md) | 5 |
| [U4.4 · Robots (parte 1)](u4-4.md) | 2 |
| [U4.5 · Robots (parte 2 y reto)](u4-5.md) | 1 |
| [U4.6 · Prueba: Cafetera y Taza](u4-6.md) | 2 |
| [U4.7 · Reto personal: cajero automático](u4-7.md) | 1 |
| [U4.8 · Cambio de rol: explícamelo tú (grupos)](u4-8.md) | 2 |
| [U5.1 · Clases abstractas, interfaces y herencia](u5-1.md) | 10 |

!!! info "Soluciones bloqueadas"
    Algunos ejercicios tienen una solución probada, **bloqueada**: solo se ve el comienzo como ejemplo. El administrador la desbloquea con el botón **🔒 Admin**.

## Equivalencias de POO en Dart

| Idea | En Dart |
|---|---|
| Constructor principal | `Persona(this.nombre, this.edad);` |
| Constructores con nombre (en vez de sobrecarga) | `Persona.sinNombre(this.edad) : nombre = '';` |
| Propiedad calculada de solo lectura | `double get imc => peso / (altura * altura);` |
| Privado | nombre que empieza por `_` (privado a la biblioteca) |
| Validar al crear | `assert(...)` o `throw ArgumentError(...)` en el cuerpo del constructor |
| Herencia | `class B extends A`, con `super.parametro` |
| Clase abstracta / interfaz | `abstract class` / `abstract interface class` |
| Enumerado | `enum Color { blanco, negro }` (admite campos y métodos) |
| Datos con igualdad por valor | sobrescribe `==` y `hashCode` (o usa el paquete `equatable`) |
| Parámetro por defecto | `{int cantidad = 200}` (con nombre) o `[int cantidad = 200]` (posicional) |
| Extensión | `extension X on List<String> { ... }` |
