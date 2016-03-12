# La Arquitectura Elm
Este tutorial esboza "La Arquitectura Elm" que verás en todos los programas escritos en [Elm][], desde [TodoMVC][] y [dreamwriter][] hasta las aplicaciones en producción en [NoRedInk][] y [CircuitHub][]. El patrón básico es útil tanto si estás escribiendo tu front-end con Elm o JS u otra cosa.

[Elm]: http://elm-lang.org/
[TodoMVC]: https://github.com/evancz/elm-todomvc
[dreamwriter]: https://github.com/rtfeldman/dreamwriter#dreamwriter
[NoRedInk]: https://www.noredink.com/
[CircuitHub]: https://www.circuithub.com/

La Arquitecutra Elm es un patrón simple para componentes anidables infinitamente. Es genial para la modularidad, reutilización del código, y pruebas. Al final, este patrón facilita crear aplicaciones web complejas de manera modular. Repasaremos 8 ejemplos, construyendo lentamente los principios y patrones fundamentales:

  1. [Contador](http://evancz.github.io/elm-architecture-tutorial/examples/1.html)
  2. [Par de contadores](http://evancz.github.io/elm-architecture-tutorial/examples/2.html)
  3. [Lista de contadores](http://evancz.github.io/elm-architecture-tutorial/examples/3.html)
  4. [Lista de contadores (variación)](http://evancz.github.io/elm-architecture-tutorial/examples/4.html)
  5. [Cargador de GIFs](http://evancz.github.io/elm-architecture-tutorial/examples/5.html)
  6. [Par de cargadores de GIFs](http://evancz.github.io/elm-architecture-tutorial/examples/6.html)
  7. [Lista de cargadores de GIFs](http://evancz.github.io/elm-architecture-tutorial/examples/7.html)
  8. [Par de cuadrados animados](http://evancz.github.io/elm-architecture-tutorial/examples/8.html)


¡Este tutorial te va a ayudar! Presentará conceptos e ideas necesarios para llegar a hacer los ejemplos 7 y 8 super fáciles. ¡Invertir tiempo en las bases merecerá la pena!

Un aspecto muy interesante de la arquitectura en todos estos programas es que *emerge* de Elm de forma natural. El diseño del propio lenguaje te lleva hacia esta arquitectura tanto si has leído este documento y conoces sus beneficios como si no. De hecho yo descubrí este patrón simplemente usando Elm y me ha dejado asombrado su simplicidad y su potencia.

**Nota**: Para seguir este tutorial con código, [instala Elm](http://elm-lang.org/install) y haz un fork de este repo. Cada ejemplo en el tutorial incluye instrucciones para ejecutarlo.

## El Patrón Básico

La lógica de cualquier programa Elm se divide entre partes claramente separadas:

* model (modelo)
* update (actualizar)
* view (vista)

Básicamente puedes empezar con el siguiente esqueleto y añadir detalles poco a poco para tu caso particular.

> Si es la primera vez que lees código Elm, echa un vistazo a la [documentación del lenguaje](http://elm-lang.org/docs) que abarca todo desde la sintaxis hasta llegar a una "mentalidad funcional". ¡Las dos primeras secciones de la [guía completa](http://elm-lang.org/docs#complete-guide) te darán soltura!

```elm
-- MODEL

type alias Model = { ... }


-- UPDATE

type Action = Reset | ...

update : Action -> Model -> Model
update action model =
  case action of
    Reset -> ...
    ...


-- VIEW

view : Model -> Html
view =
  ...
```

Este tutorial trata completamente de este patrón, y de pequeñas variaciones y ampliaciones.

## Ejemplo 1: Un contador






