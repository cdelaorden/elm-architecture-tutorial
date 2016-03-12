# La Arquitectura Elm
Este tutorial esboza "La Arquitectura Elm" que verás en todos los programas escritos en [Elm][], desde [TodoMVC][] y [dreamwriter][] hasta las aplicaciones en producción en [NoRedInk][] y [CircuitHub][]. El patrón básico es útil tanto si estás escribiendo tu front-end con Elm, JS u otra cosa.

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
  5. [Visor de GIFs aleatorios](http://evancz.github.io/elm-architecture-tutorial/examples/5.html)
  6. [Par de visorores de GIFs](http://evancz.github.io/elm-architecture-tutorial/examples/6.html)
  7. [Lista de visores de GIFs](http://evancz.github.io/elm-architecture-tutorial/examples/7.html)
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

La parte azul es el núcleo de nuestro programa Elm, que es exactamente el patrón model/update/view del que hemos estado hablando hasta ahora. Cuando programas en Elm, puedes pensar principalmente sobre el contenido de esa caja y avanzar bastante sin pensar en lo demás.

Fíjate en que no estamos **ejecutando** acciones cuando se envían de vuelta a nuestra app. Simplemente estamos enviando datos (qué queremos hacer). Esta separación es un detalle clave, ya que mantiene nuestra lógica de modificación completamente separada del código de nuestra vista.

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

Nuestro modelo es un registro con dos campos, uno para cada uno de los contadores que queremos mostrar en pantalla. Esto describe por completo todo el estado de la aplicación. Además tenemos una función `init` para crear un nuevo `Model` cuando queramos, que a su vez llama a la función `Counter.init` original para dar el valor inicial a cada uno.

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

Observa cómo podemos reutilizar la función `Counter.view` para los dos contadores. Para cada uno de ellos creamos una dirección de reenvío (con `Signal.fowardTo address Bottom`). Básicamente lo que estamos haciendo es decir, &ldquo;estos contadores etiquetan todos los mensajes salientes con `Top` o `Bottom` para que podamos distinguirlos.&rdquo;

Y eso es todo. Lo que es genial es que podemos seguir anidando más y más. Podemos coger el módulo `CounterPair`, exponer ciertos valores y funciones clave, y crear un par de pares de contadores en `CounterPairPair` o cualquier cosa que necesitemos.

### Ejemplo 3: Lista dinámica de Contadores
**[demo](http://evancz.github.io/elm-architecture-tutorial/examples/3.html) / [ver código](examples/3/)**
Un par de contadores molan, ¿pero qué me dices de una lista de contadores en la que podemos añadir y quitar contadores como queramos? ¿Nos serviría este patrón para eso también?

Una vez más podemos reutilizar el módulo `Counter` exactamente como lo dejamos en el ejercicio 1 y en el 2.

```elm
module Counter (Model, init, Action, update, view)
```

Esto implica que ya podemos empezar nuestro módulo `CounterList`. Como siempre, empezamos con nuestro modelo:

```elm
type alias Model =
    { counters : List ( ID, Counter.Model )
    , nextID : ID
    }

type alias ID = Int
```
Ahora nuestro modelo tiene una lista de contadores, cada uno identificado con un ID único. Estos IDs nos permiten distinguirlos entre sí, y así si necesitamos actualizar el contador número 4 tenemos una buena forma de hacer referencia a éste. (Este ID también nos da algo conveniente que usar como [`key`][key] (clave) cuando pensamos en optimizar el rendering, ¡pero ese no es el objetivo de este tutorial!) Nuestro modelo también guarda un `nextId` (siguiente id) que nos servirá para asignar IDs únicos a los contadores a medida que los vayamos añadiendo. 

[key]: http://package.elm-lang.org/packages/evancz/elm-html/latest/Html-Attributes#key

Ahora podemos definir el conjunto de acciones que se pueden ejecutar sobre nuestro modelo. Queremos poder añadir y eliminar contadores, y modificar contadores específicos.

```elm
type Action
    = Insert
    | Remove
    | Modify ID Counter.Action
```

Nuestro [tipo unión][] `Action` es increíblemente parecido a la descripción a alto nivel. Ya podemos escribir nuestra función `update`.

```elm
update : Action -> Model -> Model
update action model =
  case action of
    Insert ->
      let newCounter = ( model.nextID, Counter.init 0 )
          newCounters = model.counters ++ [ newCounter ]
      in
          { model |
              counters = newCounters,
              nextID = model.nextID + 1
          }

    Remove ->
      { model | counters = List.drop 1 model.counters }

    Modify id counterAction ->
      let updateCounter (counterID, counterModel) =
            if counterID == id
                then (counterID, Counter.update counterAction counterModel)
                else (counterID, counterModel)
      in
          { model | counters = List.map updateCounter model.counters }
```
Veamos una descripción por encima de cada caso:

* `Insert` &mdash; Primero creamos un nuevo contador y lo ponemos al final de nuestra lista de contadores. Después incrementamos nuestro `nextId` para tener uno listo para la siguiente vez.
  
* `Remove` &mdash; Tira el primer contador de nuestra lista.

* `Modify` &mdash; Ejecuta la función `updateCounter` (actualizar contador) sobre cada uno de nuestros contadores. Si encontramos uno con el ID indicado, ejecutamos la `Action` dada sobre éste.

Sólo nos queda definir la vista generada por la función `view`.

```elm
view : Signal.Address Action -> Model -> Html
view address model =
  let counters = List.map (viewCounter address) model.counters
      remove = button [ onClick address Remove ] [ text "Remove" ]
      insert = button [ onClick address Insert ] [ text "Add" ]
  in
      div [] ([remove, insert] ++ counters)

viewCounter : Signal.Address Action -> (ID, Counter.Model) -> Html
viewCounter address (id, model) =
  Counter.view (Signal.forwardTo address (Modify id)) model
```
La parte divertida aquí es la función `viewCounter`. Utiliza la antigua función `Counter.view`, pero en este caso proporcionamos una dirección de reenvío que anota todos los mensajes con el ID del contador particular que estamos pintando.

En la función `view` de verdad, mapeamos `viewCounter` sobre todos nuestros contadores y creamos botones de añadir y eliminar que informan a la dirección `address` directamente.

Este truco del ID se puede emplear siempre que quieras un número dinámico de subcomponentes. Los contadores son muy básicos, pero este patrón funcionaría exactamente igual si tuvieras una lista de perfiles de usuarios, tweets, noticias o detalles de productos.

### Ejemplo 4: Una lista más guay de Contadores
**[demo](http://evancz.github.io/elm-architecture-tutorial/examples/4.html) / [ver código](examples/4/)**

Vale, mantener las cosas simples y modulares con una lista dinámica de contadores es genial, pero en lugar de un botón general de eliminar, ¿qué pasaría si cada contador tuviera su botón específico de eliminar? ¡Seguro que *eso* nos va complicar las cosas!

Nah, funciona.

En este caso nuestro objetivo implica que necesitamos una manera de mostrar un `Counter` con un botón de eliminar. Curiosamente, podemos mantener la función `view` de la versión anterior y añadir una nueva función `viewWithRemoveButton` (vista con botón eliminar) al módulo `Counter` que proporcione una vista ligeramente diferente de nuestro `Model` subyacente. Es bastante guay. No necesitamos duplicar código o hacer locuras con tipos derivados o sobrecargas. ¡Simplemente añadimos una nueva función a la API pública para exponer la nueva funcionalidad!

```elm
module Counter (Model, init, Action, update, view, viewWithRemoveButton, Context) where

...

type alias Context =
    { actions : Signal.Address Action
    , remove : Signal.Address ()
    }

viewWithRemoveButton : Context -> Model -> Html
viewWithRemoveButton context model =
  div []
    [ button [ onClick context.actions Decrement ] [ text "-" ]
    , div [ countStyle ] [ text (toString model) ]
    , button [ onClick context.actions Increment ] [ text "+" ]
    , div [ countStyle ] []
    , button [ onClick context.remove () ] [ text "X" ]
    ]
```
La función `viewWithRemoveButton` añade el botón extra. Observa que los botones incrementar/decrementar envían mensajes a la dirección `actions` pero el de eliminar los envía a la dirección `remove`. Estos mensajes que enviamos a `remove` básicamente dicen, &ldquo;¡oye, a quien pertenezca, elimíname!&rdquo; Es tarea de quien quiera que sea dueño de ese contador particular llevar a cabo la eliminación.

Ahora que ya tenemos nuestra nueva función `viewWithRemoveButton`, podemos crear un módulo `CounterList` que incluya todos los contadores individuales. El `Model` es idéntico al del ejemplo 3: una lista de contadores y un ID único.


```elm
type alias Model =
    { counters : List ( ID, Counter.Model )
    , nextID : ID
    }

type alias ID = Int
```
Nuestras acciones son un poco diferentes. En lugar de eliminar cualquier contador, queremos eliminar uno específico, por lo que el caso `Remove` ahora tiene un ID.

```elm
type Action
    = Insert
    | Remove ID
    | Modify ID Counter.Action
```

La función `update` es muy similar a la del ejemplo 3 también.

```elm
update : Action -> Model -> Model
update action model =
  case action of
    Insert ->
      { model |
          counters = ( model.nextID, Counter.init 0 ) :: model.counters,
          nextID = model.nextID + 1
      }

    Remove id ->
      { model |
          counters = List.filter (\(counterID, _) -> counterID /= id) model.counters
      }

    Modify id counterAction ->
      let updateCounter (counterID, counterModel) =
            if counterID == id
                then (counterID, Counter.update counterAction counterModel)
                else (counterID, counterModel)
      in
          { model | counters = List.map updateCounter model.counters }
```
En el caso de `Remove`, sacamos el contador cuyo ID coincide con el que tenemos que eliminar, filtrando la lista con una función anónima definida en línea que devuelve verdadero si el ID del elemento es *distinto* al ID que queremos eliminar. El resto de casos son casi iguales a cómo eran antes.

Para terminar, unimos todas las piezas en la función `view`:

```elm
view : Signal.Address Action -> Model -> Html
view address model =
  let insert = button [ onClick address Insert ] [ text "Add" ]
  in
      div [] (insert :: List.map (viewCounter address) model.counters)

viewCounter : Signal.Address Action -> (ID, Counter.Model) -> Html
viewCounter address (id, model) =
  let context =
        Counter.Context
          (Signal.forwardTo address (Modify id))
          (Signal.forwardTo address (always (Remove id)))
  in
      Counter.viewWithRemoveButton context model
```

En la función `viewCounter`, construimos el contexto (`Counter.Context`) para pasar las direcciones de reenvío necesarias. En ambos casos anotamos cada `Counter.Action` para que sepamos cuál debemos modificar o eliminar.

## Lecciones Importantes Hasta Ahora
**Patrón básico** &mdash; Todo se construye alrededor de un `Model`, una manera de actualizar (`update`) ese modelo, y una manera de ver (`view`) ese modelo. Todo es una variación de este patrón básico.

**Anidar Módulos** &mdash; Las direcciones de reenvío (forward) nos facilitan anidar nuestro patrón básico, ocultando la implementación por completo. Podemos anidar este patrón tan profundo como queramos, y cada nivel sólo necesita saber qué pasa con el nivel inferior siguiente.

**Añadir Contexto** &mdash; A veces para actualizar o ver nuestro modelo (funciones `update` y `view`), se necesita información adicional. Siempre podemos añadir algo de contexto (`Context`) a estas funciones y pasarle toda la información necesaria sin complicar nuestro `Model`.

```elm
update : Context -> Action -> Model -> Model
view : Context' -> Model -> Html
```

En cada nivel de profundidad podemos derivar el `Context` específico necesario para cada submódulo.

**Testing sencillo** &mdash; Todas las funciones que hemos creado son [funciones puras][pure]. Esto hace que sea extremadamente fácil probar tu función `update`. No hace falta inicializar, simular ni configurar nada, simplemente llamas a la función con los argumentos que quieras testar.

[pure]: http://en.wikipedia.org/wiki/Pure_function

## Ejemplo 5: Visor de GIFs aleatorios

**[demo](http://evancz.github.io/elm-architecture-tutorial/examples/5.html) / [ver código](examples/5/)**

Ya hemos visto cómo crear componentes y anidarlos cuanto queramos, ¿pero qué ocurre si queremos hacer una petición HTTP a alguien ahí fuera? ¿O comunicarnos con una base de datos? Este ejemplo empieza utilizando el [paquete `elm-effects`][fx] para crear un componente sencillo que trae GIFs aleatorios de giphy.com con el tema "funny cats" (gatos divertidos).

[fx]: http://package.elm-lang.org/packages/evancz/elm-effects/latest

Cuando mires [la implementación](examples/5/RandomGif.elm), observa que es prácticamente el mismo código que el contador del ejemplo 1. El `Model` es muy típico:

```elm
type alias Model =
    { topic : String
    , gifUrl : String
    }
```
Tenemos que saber cual es el tema (`topic`) del visor y la URL del GIF (`gifUrl`) que estamos mostrando en cada momento. La única diferencia en este ejemplo es que las funciones `init` y `update` tienen tipos más elaborados:

```elm
init : String -> (Model, Effects Action)

update : Action -> Model -> (Model, Effects Action)
```
En lugar de devolver solo un nuevo `Model` también devolvemos algunos efectos que nos gustaría ejecutar. Así que utilizaremos la [API de `Effects`][fx_api], que es más o menos algo como esto:

[fx_api]: http://package.elm-lang.org/packages/evancz/elm-effects/latest/Effects

```elm
module Effects where

type Effects a

none : Effects a
  -- don't do anything

task : Task Never a -> Effects a
  -- request a task, do HTTP and database stuff
```

El tipo `Effects` es básicamente una estructura de datos que guarda un montón de tareas (task) independientes que serán ejecutadas en algún momento posterior. Vamos a coger una sensación mejor de cómo funciona comprobando lo que hace `update` en este ejemplo:

```elm
type Action
    = RequestMore
    | NewGif (Maybe String)


update : Action -> Model -> (Model, Effects Action)
update msg model =
  case msg of
    RequestMore ->
      ( model
      , getRandomGif model.topic
      )

    NewGif maybeUrl ->
      ( Model model.topic (Maybe.withDefault model.gifUrl maybeUrl)
      , Effects.none
      )

-- getRandomGif : String -> Effects Action
```

El usuario puede lanzar una acción `RequestMore` (pedir más) haciendo clic en el botón apropiado, y cuando el servidor responda lo hará mediante una acción `NewGif`. Manejamos ambos casos en nuestra función `update`.

En el caso de `RequestMore` primero devolvemos el modelo existente. El usuario ha hecho clic en un botón, no hay nada que cambie en este momento. Además creamos una `Effects Action` llamando a la función `getRandomGif` con el tema a buscar. Veremos esta función pronto. Por ahora sólo nos hace falta saber que cuando se ejecuta una `Effects Action`, generará un montón de valores `Action` que pasarán por nuestra aplicación. Al final, `getRandomGif model.topic` acabará resultando en una acción como esta:

```elm
NewGif (Just "http://s3.amazonaws.com/giphygifs/media/ka1aeBvFCSLD2/giphy.gif")
```
Devuelve un `Maybe` (quizá) porque la petición al servidor puede fallar. Esa acción será recibida de nuevo por nuestra función `update`. Así que cuando el código vaya por la ruta `NewGif` simplemente actualizamos en el modelo la `gifUrl` actual, si es posible. Si la petición falló, mantenemos la misma `model.gifUrl`.

Algo parecido ocurre en `init` que define el modelo inicial y solicita una GIF del tema correcto a la API de giphy.com.

```elm
init : String -> (Model, Effects Action)
init topic =
  ( Model topic "assets/waiting.gif"
  , getRandomGif topic
  )

-- getRandomGif : String -> Effects Action
```
Como hemos dicho, cuando el efecto del GIF aleatorio termine, producirá una `Action` que será recibida por nuestra función `update`.

> **Nota:** Hasta ahora hemos estado utilizando el módulo `StartApp.Simple` del [paquete start-app](http://package.elm-lang.org/packages/evancz/start-app/latest), pero esta vez lo actualizamos al módulo `StartApp`, que es capaz de gestionar la complejidad de apps web más realistas. Tiene una [API ligeramente más elaborada](http://package.elm-lang.org/packages/evancz/start-app/latest/StartApp). El cambio crucial es que puede manejar nuestros nuevos tipos devueltos en `init` y `update`.

Uno de los aspectos cruciales de este ejemplo es la función `getRandomGif` que realmente describe cómo obtener un GIF aleatorio. Utiliza [tasks][] (tareas) y el [paquete `Http`][http], y voy a intentar ofrecer una visión general de cómo se usan estas cosas a medida que lo vayamos viendo. Veamos el código:

[tasks]: http://elm-lang.org/guide/reactivity#tasks
[http]: http://package.elm-lang.org/packages/evancz/elm-http/latest

```elm
getRandomGif : String -> Effects Action
getRandomGif topic =
  Http.get decodeImageUrl (randomUrl topic)
    |> Task.toMaybe
    |> Task.map NewGif
    |> Effects.task

-- La primera línea ha creado una petición HTTP GET. Intenta
-- traer JSON en la dirección `randomUrl topic` y decodifica 
-- el resultado con `decodeImageUrl`. Las dos están definidas
-- más abajo.

-- Después usamos `Task.toMaybe` para capturar posibles fallos y
-- aplicamos la etiqueta `NewGif` para convertir el resultado 
-- en una `Action`
-- Para terminar lo convertimos en un valor `Effects` que podemos
-- emplear en las funciones `init` o `update`

-- Dado un tema, construye una URL para la API de giphy.com
randomUrl : String -> String
randomUrl topic =
  Http.url "http://api.giphy.com/v1/gifs/random"
    [ "api_key" => "dc6zaTOxFJmzC"
    , "tag" => topic
    ]


-- Un decodificador JSPN que recibe un gran fragmento de datos
-- desde giphy y extrae la cadena en `json.data.image_url`
decodeImageUrl : Json.Decoder String
decodeImageUrl =
  Json.at ["data", "image_url"] Json.string
```
Una vez hemos escrito esto, ya podemos reutilizar `getRandomGif` en nuestras funciones `init` y `update`.

Algo interesante sobre la tarea devuelta por `getRandomGif` es que no puede fallar `Never` (nunca). La idea es que cualquier posible fallo *tiene que* ser manejado de forma explícita. No queremos tareas que fallen silenciosamente.

Voy a tratar de explicar exactamente cómo funciona eso, pero no es imprescindible comprender todos los detalles para utilizar estas cosas. Veamos, cada `Task` tiene un tipo "éxito" y un tipo "fallo". Por ejemplo, una tarea HTTP puede tener un tipo como `Task Http.Error String` tal que fallará con un `Http.Error` o terminará con éxito con un `String`. Esto hace que podamos encadenar de forma agradable varias tareas juntas sin preocuparnos demasiado por los errores. Supongamos que nuestro componente lanza una tarea, pero ésta falla. ¿Qué ocurre entonces? ¿Quién se entera? ¿Cómo nos recuperamos del error? Al hacer el tipo fallo `Never` forzamos que cualquier error vaya en el tipo "éxito" de forma que puede ser gestionado de forma explícita en el componente. En nuestro caso, usamos `Task.toMaybe` para que nuestra función `update` gestione los fallos de HTTP (con el `Maybe.withDefault` para dar un valor por defecto en caso de error). Esto significa que las tareas no pueden fallar sin que nos enteremos, ya que gestionamos los posibles errores manualmente, de forma explícita.

### Ejemplo 6: Par de visores de GIFs aleatorios
**[demo](http://evancz.github.io/elm-architecture-tutorial/examples/6.html) / [ver código](examples/6/)**

De acuerdo, podemos hacer efectos, pero ¿qué hay de efectos *anidados*? ¿Pensaste en ese caso? Esta versión utiliza el código intacto del ejemplo 5 para crear un par de visores de GIFs independientes.

Cuando leas [la implementación](examples/6/RandomGifPair.elm), observa que es muy similar al código del par de contadores del ejemplo 2. El `Model` se define con dos valores de tipo `RandomGif.Model`:

```elm
type alias Model =
    { left : RandomGif.Model
    , right : RandomGif.Model
    }
```

Esto nos permite gestionar cada uno de forma independiente. Por tanto, nuestras acciones simplemente encaminan los mensajes al subcomponente apropiado.

```elm
type Action
    = Left RandomGif.Action
    | Right RandomGif.Action
```

Los interesante es que utilizamos realmente las etiquetas `Left` y `Right` un poco en nuestras funciones `update` e `init`.

```elm
-- Effects.map : (a -> b) -> Effects a -> Effects b

update : Action -> Model -> (Model, Effects Action)
update action model =
  case action of
    Left msg ->
      let
        (left, fx) = RandomGif.update msg model.left
      in
        ( Model left model.right
        , Effects.map Left fx
        )

    Right msg ->
      let
        (right, fx) = RandomGif.update msg model.right
      in
        ( Model model.left right
        , Effects.map Right fx
        )
```

En cada rama llamamos a `RandomGif.update` (la del ejemplo anterior) que nos devuelve un nuevo modelo y algunos efectos que llamamos `fx`. Devolvemos un modelo actualizado como siempre, pero tenemos que hacer trabajo extra sobre nuestros efectos. En lugar de devolverlos directamente, utilizamos la función [`Effects.map`](http://package.elm-lang.org/packages/evancz/elm-effects/latest/Effects#map) para transformarlos en el mismo tipo de `Action`. Esto funciona muy parecido a `Signal.forwardTo`, permitiéndonos etiquetar los valores para clarificar cómo deberían ser atendidos.

Lo mismo ocurre en la función `init`. Proporcionamos un tema para cada visor de GIFs aleatorios y obtenemos un modelo inicial y efectos, para cada uno de ellos.

```elm
init : String -> String -> (Model, Effects Action)
init leftTopic rightTopic =
  let
    (left, leftFx) = RandomGif.init leftTopic
    (right, rightFx) = RandomGif.init rightTopic
  in
    ( Model left right
    , Effects.batch
        [ Effects.map Left leftFx
        , Effects.map Right rightFx
        ]
    )

-- Effects.batch : List (Effects a) -> Effects a
```

En este caso no solo utilizamos `Effects.map` para etiquetar los resultados, sino también la función [`Effects.batch`](http://package.elm-lang.org/packages/evancz/elm-effects/latest/Effects#batch) para meter todos es un mismo saco. Todas las tareas solicitadas serán ejecutadas de forma independiente, y por tanto los efectos de `right` y `left` avanzarán simultáneamente.


## Ejemplo 7: Lista de visores de GIFs aleatorios

**[demo](http://evancz.github.io/elm-architecture-tutorial/examples/7.html) / [ver código](examples/7/)**

Este ejemplo te permite tener una lista de visores de GIFs aleatorios en la que puedes establecer los temas tú mismo. De nuevo, reutilizaremos el módulo base `RandomGif` tal cual.

En [la implementación](examples/7/RandomGifList.elm) verás que se corresponde exactamente con el ejemplo 3. Colocamos todos nuestros submodelos en una lista, asociados a un ID y realizamos las operaciones basándonos en esos IDs. La única novedad es que utilizamos `Effects` en las funciones `init` y `update`, agrupándolos con `Effects.map` y `Effects.batch` 

Por favor, crea un issue si esta sección debería dar más detalles de cómo funcionan las cosas.

## Ejemplo 8: Animación
**[demo](http://evancz.github.io/elm-architecture-tutorial/examples/8.html) / [ver código](examples/8/)**

Ya hemos visto componentes con tareas que pueden componerse unos dentro de otros de cualquier manera pero, ¿cómo funciona esto con animaciones?

Curiosamente, ¡es prácticamente igual! (O quizá ya no sea tan sorprendente que el mismo patrón de todos los demás ejercicios funcione ahora también... ¡Parece un buen patrón!)

Este ejemplo es un par de cuadrados sobre los que puede hacer clic. Cuando haces clic en un cuadrado, rota 90 grados. El esquema es una forma adaptada del ejemplo 2 (par de contadores) y del ejemplo 6 (par de visores), escribiendo toda la lógica de la animación en `SpinSquare.elm` que luego reutilizamos en `SpinSquarePair.elm`.

Puesto que todas las cosas interesantes y novedosas ocurren [en `SpinSquare`](examples/8/SpinSquare.elm), vamos a concentrarnos en ese código. Lo primero que necesitamos es un modelo:

```elm
type alias Model =
    { angle : Float
    , animationState : AnimationState
    }


type alias AnimationState =
    Maybe { prevClockTime : Time,  elapsedTime: Time }


rotateStep = 90
duration = second
```

Nuestro modelo fundamental es el `angle` (ángulo) en el que está actualmente el cuadrado y un `animationState` que guarda qué está ocurriendo con una animación empezada. Si no hay ninguna es `Nothing`, pero si está en marcha contiene:

* `prevClockTime` &mdash; El tiempo de reloj más reciente que emplearemos para calcular los diferenciales de tiempo. Nos ayudará a saber exactamente cuántos milisegundos han pasado desde el último fotograma.
* `elapsedTime` &mdash; Un número entre 0 y `duration` que nos indica el progreso en la animación.

La constante `rotateStep` simplemente define cuánto giramos el cuadrado con cada clic. Puedes cambiar este valor y todo funcionará igual.

Lo verdadermanete interesante ocurre en `update`:

```elm
type Action
    = Spin
    | Tick Time


update : Action -> Model -> (Model, Effects Action)
update msg model =
  case msg of
    Spin ->
      case model.animationState of
        Nothing ->
          ( model, Effects.tick Tick )

        Just _ ->
          ( model, Effects.none )

    Tick clockTime ->
      let
        newElapsedTime =
          case model.animationState of
            Nothing ->
              0

            Just {elapsedTime, prevClockTime} ->
              elapsedTime + (clockTime - prevClockTime)
      in
        if newElapsedTime > duration then
          ( { angle = model.angle + rotateStep
            , animationState = Nothing
            }
          , Effects.none
          )
        else
          ( { angle = model.angle
            , animationState = Just { elapsedTime = newElapsedTime, prevClockTime = clockTime }
            }
          , Effects.tick Tick
          )
```

Hay dos tipos de `Action` que tenemos que gestionar:

- `Spin` (girar) indica que el usuario ha hecho clic en el cuadrado, solicitando el giro. Por tanto en `update` pedimos un nuevo tick de reloj si no había animación en marcha o dejamos todo igual si ya había una empezada.
- `Tick` indica que ha pasado un ciclo de reloj por lo que tenemos que avanzar un paso la animación, por lo que en `update` tenemos que modificar el estado de la animación (`animationState`) en nuestro modelo. Primero, comprobamos si hay alguna animación en marcha. Si es así, calculamos el nuevo tiempo transcurrido `newElapsedTime` sumando el tiempo de reloj actual y restando el anterior. Si este nuevo tiempo transcurrido es mayor que la duración total `duration` paramos la animación y dejamos de pedir ciclos de reloj. De lo contrario, actualizamos el estado de la animación y solicitamos otro tick.

Una vez más, creo que podemos recortar este código a medida que escribamos más código parecido y veamos aparecer el patrón general. ¡Debería ser divertido encontrarlo!

Para terminar tenemos una función `view` un tanto interesante. Este ejemplo nos muestra una llamativa animación con rebote, pero simplemente estamos incrementando nuestro `elapsedTime` en el modelo en tramos lineares. ¿Cómo ocurre eso?

El código de `view` es [`elm-svg`](http://package.elm-lang.org/packages/evancz/elm-svg/latest/) estándar para generar formas clicables un poco elaboradas. La parte interesante del código de la vista es `toOffset`, que calcula la rotación a aplicar para el `AnimationState` actual.

```elm
-- import Easing exposing (ease, easeOutBounce, float)

toOffset : AnimationState -> Float
toOffset animationState =
  case animationState of
    Nothing ->
      0

    Just {elapsedTime} ->
      ease easeOutBounce float 0 rotateStep duration elapsedTime
```

Estamos utilizando el [paquete de easing](http://package.elm-lang.org/packages/Dandandan/Easing/latest) de [@Dandandan](https://github.com/Dandandan), que nos permite aplicar fácilmente todo tipo de funciones de suavizado ([easings]((http://easings.net/))) sobre números, colores, puntos o cualquier otra cosa loca que se te ocurra.

La función `ease` toma un número entre 0 y `duration`, y lo convierte a un número entre 0 y `rotateStep` que hemos establecido a 90 grados al comienzo del programa. Además proporcionamos un suavizado. En nuestro caso le pasamos `easeOutBounce` lo que significa que cuando vayamos avanzando entre 0 y `duration`, obtendremos un número entre 0 y 90 con ese suavizado aplicado. ¡Alucinante! ¡Prueba a cambiar `easeOutBounce` por algún otro [easing](http://package.elm-lang.org/packages/Dandandan/Easing/latest/Easing) y comprueba cómo queda!

Para tener los dos cuadrados, conectamos todo en `SpinSquarePair`, pero ese código es casi idéntico al de los otros ejemplos de pares, el 2 y el 6.

¡Muy bien, esos son los primeros pasos para hacer animaciones con esta librería! No está claro que hayamos dado en el clavo con todo aquí, así que haznos saber cómo van las cosas a medida que ganes experiencia. Con un poco de suerte podremos hacerlo incluso más fácil.

> **Nota:** Espero que podamos construir algunas abstracciones sobre las ideas clave expuestas aquí. Este ejemplo realiza algunas tareas de bajo nivel, pero apuesto a que podemos encontrar algunos patrones decentes para facilitarlo a medida que trabajemos más en ello. Si te parece raro, ¡intenta hacer algo mejor y cuéntanoslo!

