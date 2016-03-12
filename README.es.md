# La Arquitectura Elm
Este tutorial esboza "La Arquitectura Elm" que verás en todos los programas escritos en [Elm][], desde [TodoMVC][] y [dreamwriter][] hasta las aplicaciones en producción en [NoRedInk][] y [CircuitHub][]. El patrón básico es útil tanto si estás escribiendo tu front-end con Elm o JS u otra cosa.

[Elm]: http://elm-lang.org/
[TodoMVC]: https://github.com/evancz/elm-todomvc
[dreamwriter]: https://github.com/rtfeldman/dreamwriter#dreamwriter
[NoRedInk]: https://www.noredink.com/
[CircuitHub]: https://www.circuithub.com/

La Arquitectura Elm es un patrón simple para componentes anidables infinitamente. Es genial para la modularidad, reutilización del código, y pruebas. Al final, este patrón facilita crear aplicaciones web complejas de manera modular. Repasaremos 8 ejemplos, construyendo lentamente los principios y patrones fundamentales:

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

**[demo](http://evancz.github.io/elm-architecture-tutorial/examples/1.html) / [ver código](examples/1/)**

Nuestro primer ejemplo es un simple contador que se puede incrementar o decrementar.

[El código](examples/1/Counter.elm) empieza con un modelo muy simple. Sólo tenemos que llevar la cuenta de un único número:

```elm
type alias Model = Int
```

Si queremos actualizar nuestro modelo, vuelve a ser relativamente simple. Definimos una serie de acciones que se pueden llevar a cabo, y la función `update` que ejecuta realmente esas acciones:

```elm
type Action = Increment | Decrement

update: Action -> Model -> Model
update action model =
    case action of
        Increment -> model + 1
        Decrement -> model - 1
```

Fíjate que nuestro [tipo unión][] `Action` no *hace* nada. Simplemente describe las acciones posibles. Si alguien decide que nuestro contador debería doblarse cuando se pulsa cierto botón, ese sería un nuevo caso en `Action`. Esto significa que nuestro código acaba siendo muy claro en cuanto a cómo se puede transformar nuestro modelo. Cualquiera que lea este código sabrá inmediatamente lo que se permite y lo que no. Además, sabrán exactamente como añadir funcionalidad de forma consistente.

[tipo unión]: http://elm-lang.org/learn/Union-Types.elm

Finalmente, creamos una forma de ver (`view`) nuestro `Model`. Estamos usando [elm-html][] para crear un poco de HTML para mostrar en el browser. Vamos a crear un div que contiene: un botón de decrementar, un div que muestra la cuenta actual, y un botón de incrementar.

[elm-html]: http://elm-lang.org/blog/Blazing-Fast-Html.elm

```elm
view : Signal.Address Action -> Model -> Html
view address model =
  div []
    [ button [ onClick address Decrement ] [ text "-" ]
    , div [ countStyle ] [ text (toString model) ]
    , button [ onClick address Increment ] [ text "+" ]
    ]

countStyle : Attribute
countStyle =
  ...
```

La parte difícil de nuestra función `view` es la `Address` (dirección). ¡Entraremos de lleno en eso en la siguiente sección! Por ahora, sólo quiero que te des cuenta de que **este código es completamente declarativo**. Recibimos un `Model` y generamos un poco de `Html`. Eso es todo. En ningún momento modificamos el DOM manualmente, lo cual le da a la librería [mucha más libertad para hacer optimizaciones inteligentes][elm-html] y en realidad hace el rendering *más rápido* al final. Es de locos. Además, `view` es simplemente una función con lo que tenemos acceso a toda la potencia del sistema de módulos, los frameworks de test y las librerías de Elm al crear vistas.

Este patrón es la esencia del diseño de programas Elm. Cada ejemplo que veamos de ahora en adelante será una ligera variación de este patrón básico: `Model`, `update`, `view`.

## Iniciar el programa
Prácticamente todos los programas Elm tendrán un pequeño código que maneja toda la aplicación. Para cada ejemplo en este tutorial, ese código está separado en el archivo `Main.elm`. En nuestro ejemplo del contador, la parte interesante es así:

```elm
import Counter exposing (update, view)
import StartApp.Simple exposing (start)

main =
  start { model = 0, update = update, view = view }
```

Estamos usando el paquete [`StartApp`](https://github.com/evancz/start-app) para conectar nuestro modelo con las funciones de actualización y vista. Es un pequeño envoltorio sobre [señales de Elm](http://elm-lang.org/learn/Using-Signals.elm) para que no tengas que entrar en ese concepto todavía.

La clave para interconectar nuestra aplicación es el concepto de `Address` (dirección). Cada manejador de eventos en nuestra función `view` informa a una dirección específica. Simplemente le envía fragmentos de datos. El paquete `StartApp` monitoriza todos los mensajes que llegan a esta dirección y se los inyecta a la función `update`. El modelo se actualiza y [elm-html][] se encarga de representar los cambios de manera eficiente.

Esto significa que los valores fluyen en un programa Elm en una única dirección, algo como esto:

![Resumen del grafo de señales](diagrams/signal-graph-summary.png)

La parte azul es el núcleo de nuestro programa Elm, que es exactamente el patrón model/update/view del que hemos estado hablando hasta ahora. Cuando programas en Elm, puedes pensar principalmente sobre esta caja y realizar grandes progresos.

Fíjate en que no estamos **realizando** acciones cuando son enviadas de vuelta a nuestra app. Simplemente estamos enviando datos. Esta separación es un detalle clave, ya que mantiene nuestra lógica completamente separada del código de nuestra vista.

## Ejemplo 2: Par de contadores
**[demo](http://evancz.github.io/elm-architecture-tutorial/examples/2.html) / [ver código](examples/2/)**

En el ejemplo 1 creamos un contador básico pero ¿cómo escala este patrón si queremos *dos* contadores? ¿Podemos mantenerlo todo modularizado?

¿No sería genial si pudiésemos reutilizar todo el código del ejemplo 1? Lo sorprendente de la Arquitectura Elm es que **podemos reutilizar código sin cambiar absolutamente nada**. Cuando creamos el módulo `Counter` en el ejemplo anterior, encapsulamos todos los detalles de implementación así que podemos utilizarlo en cualquier otro sitio:

```elm
module Counter (Model, init, Action, update, view) where

type Model

init : Int -> Model

type Action

update : Action -> Model -> Model

view : Signal.Address Action -> Model -> Html
```

Escribir código modular tiene que ver con crear abstracciones sólidas. Queremos fronteras que expongan funcionalidad pero oculten la implementación de forma apropiada. Desde fuera del módulo `Counter`, sólo vemos un conjunto básico de valores: `Model`, `init`, `Action`, `update` y `view`. No nos interesa cómo están implementadas esas cosas. De hecho, es **imposible** saber cómo están implementadas. Esto implica que nadie puede depender de los detalles de implementación que no se han hecho públicos.

Así que podemos reutilizar nuestro módulo `Counter`, pero ahora lo usaremos para crear nuestro `CounterPair` (par de contadores). Como siempre, comenzamos con un `Model`:

```elm
type alias Model =
    { topCounter : Counter.Model
    , bottomCounter : Counter.Model
    }

init : Int -> Int -> Model
init top bottom =
    { topCounter = Counter.init top
    , bottomCounter = Counter.init bottom
    }
```

Nuestro modelo es un registro con dos campos, uno para cada uno de los contadores que queremos mostrar en pantalla. Esto describe por completo todo el estado de la aplicación. Además tenemos una función `init` para crear un nuevo `Model` cuando queramos.

Después describimos el conjunto de **acciones** que queremos soportar. Esta vez nuestras características deberían ser: reiniciar todos los contadores, actualizar el contador superior (`topCounter`) o actualizar el inferior (`bottomCounter`).

```elm
type Action
    = Reset
    | Top Counter.Action
    | Bottom Counter.Action
```

Fíjate que nuestro [tipo unión][] hace referencia al tipo `Counter.Action`, pero no sabemos los detalles de esas acciones. Cuando escribamos nuestra función `update`, estamos principalmente dirigiendo esas `Counter.Actions` al sitio correcto:

```elm
update: Action -> Model -> Model
update action model =
  case action of  
    Reset -> init 0 0

    Top act ->
      { model | 
          topCounter = Counter.update act model.topCounter
      }

    Bottom act ->
      { model |
          bottomCounter = Counter.update act model.bottomCounter  
      }
```

Y por último nos falta escribir una función `view` que muestre ambos contadores en pantalla, junto a un botón de reinicio.

```elm
view : Signal.Address Action -> Model -> Html
view address model =
  div []
    [ Counter.view (Signal.forwardTo address Top) model.topCounter
    , Counter.view (Signal.forwardTo address Bottom) model.bottomCounter
    , button [ onClick address Reset ] [ text "RESET" ]
    ]
```

Observa cómo podemos reutilizar la función `Counter.view` para los dos contadores. Para cada uno de ellos creamos una dirección de reenvío. Básicamente lo que estamos haciendo es decir, &ldquo;estos contadores etiquetan todos los mensajes salientes con `Top` o `Bottom` para que podamos distinguirlos.&rdquo;

Y eso es todo. Lo que es genial es que podemos seguir anidando mñas y más. Podemos coger el módulo `CounterPair`, exponer ciertos valores y funciones clave, y crear un par de pares de contadores en `CounterPairPair` o cualquier cosa que necesitemos.









