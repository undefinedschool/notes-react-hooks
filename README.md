# React Hooks [WIP]

### 👉 Ver [todas las notas](https://github.com/undefinedschool/notes)

## Por qué usar Hooks?

Una de las últimas features importantes que React introdujo fueron los [Hooks](https://es.reactjs.org/blog/2019/02/06/react-v16.8.0.html), que, en un muy resumido resumen, nos permiten manejar estado dentro de [_componentes funcionales_](https://github.com/undefinedschool/notes-react-basics#functional-o-stateless-components), algo para lo cual antes necesitábamos utilizar [_Class Components_](https://github.com/undefinedschool/notes-react-basics#class-o-stateful-components) si o si. De esta forma, nuestro código (y el manejo del state) se simplifican muchísimo :D

Algunos inconvenientes que tenemos en React utilizando clases, son, por ejemplo

- **Constructor**: este no es un problema particular de React sino de cómo funcionan las [clases en JS](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes). Los _Class Components_ de React siempre extienden `React.Component`. En las clases de JS, para poder utilizar `this`, tenemos que invocar primero al constructor de la superclase (la que estamos extendiendo o de la que estamos 'heredando'), usando `super`. En el caso de React en particular, tenemos que acordarnos siempre de llamar a `super` pasándole `props` (las [_props_ ](https://github.com/undefinedschool/notes-react-basics#props) que recibe el componente) antes de poder setear el estado.

```js
constructor(props) {
  super(props);
  
  this.state = {
    ...
  };
}
```

- **No autobinding**: otro inconveniente que tenemos al usar Class Components es con la referencia a `this` en los métodos que modifican el state de los mismos. Por ejemplo, si le pasamos un método de un componente como prop a otro (un caso super común es, por ejemplo, cambiar el state de un component cuando se clickea un botón, definido como _child component_ del mismo), React en principio no sabe cuál es el contexto de `this`, se pierde la referencia original, por lo que tenemos que acordarnos también de [_bindear_ estos métodos en el constructor](https://stackoverflow.com/questions/32317154/react-uncaught-typeerror-cannot-read-property-setstate-of-undefined), agregando boilerplate al código.

```js
constructor(props) {
  super(props);
  
  this.state = {
    ...
  };

  // binding
  this.setText = this.setText.bind(this);
}
```

> **Nota:** los [_Class fields_](https://blog.g2i.co/react-class-components-with-es6-and-class-fields-927b2b59f94e) que se agregaron después a las clases de JS resuelven estos problemas mencionados, pero no es muy común su uso 

- **Component lifecycle**: la forma en que funciona el [_ciclo de vida_ de los componentes en React](https://reactjs.org/docs/state-and-lifecycle.html) nos lleva a estructurar los componentes de cierta forma, duplicando mucha lógica de los mismos, por ejemplo al considerar diferentes _side-effects_.

- **Lógica compartida no relacionada a la UI**: siempre pensamos a React como una librería para construir interfaces, en la que componemos diferentes componente que representan partes de esta UI. Pero estos componentes muchas veces pueden incluir lógica que no necesariamente tenga que ver con la UI y podemos necesitar compartirla con otros, duplicando nuevamente el código. Una solución a esto fueron los [_Higher-Order Components_](https://reactjs.org/docs/higher-order-components.html) (aka _HOCs_), funciones que reciben un componente y retornan uno nuevo. El problema con esta solución es que la lógica puede volverse bastante compleja de seguir si empezamos a componer diferentes HOCs.

```js
// HOCs wrapper hell
export default withHover {
  withTheme(
    withAuth(
      withRepos(UserProfile)
    )
  )
};
```

### ft. React Hooks

Acá es donde aparecen los famosos Hooks, que poponen una nueva API para resolver estos problemas, de una forma mucho más simple y usando sólo... **funciones** 😎.

[![](https://i.imgur.com/kOUapFS.png)](https://twitter.com/id_aa_carmack/status/53512300451201024)

## Algunos de los Hooks más comunes

### Estado del componente: [`useState`](https://reactjs.org/docs/hooks-state.html)

> _tl;dr_ nos permite preservar valores entre renders y triggerear un re-render del componente

Este Hook nos permite crear componentes funcionales que puedan manejar su propio _state_. `useState` es una función que retorna un _array_ con 2 valores (también se le dice tupla), el 1ro representa el state del componente y el 2do, una función que nos provee el hook para actualizar el state. Además, recibe un valor para inicializar el state.

En el siguiente ejemplo, el valor inicial del state (`text`) es un string vacío y `setText` es la función que usaremos para actualizarlo.

```js
import React, { useState } from 'react';

function App() {
  // usamos destructuring para obtener los valores del array que retorna `useState`
  const [text, setText] = useState('0 clicks given.');

  return <button onClick={() => setText('clicked!')}>{text}</button>;
}
```

> **Nota:** por convención, al método que utilizamos para actualizar el estado se le suele poner el prefijo _set_ (ej: si el state es `text`, el método será `setState`)

Es muy importante tener en cuenta que, a diferencia de lo que pasa con `this.setState()` en un Class Componente, **el valor que recibe la funcion para actualizar el state** (también conocida como _state dispatch function_) **no se mergea con el state actual, sino que lo sobreeescribe**. Es por esto que, en el caso de querer representar el valor del state con un objeto por ejemplo, en lugar de valores individuales, es mejor utilizar el hook [`useReducer`](https://github.com/undefinedschool/notes-react-hooks#usereducer).

El ejemplo anterior, usando clases (Class Components) se escribiría de la siguiente forma 

```js
import React from 'react';

class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      text = '0 clicks given.'
    };
    
    this.setText = this.setText.bind(this);
  }

  setText(newText) {
    this.setState({
      text: newText
    });
  }

  render() {
    return <button onClick={() => setText('clicked!')}>{text}</button>;
  }
}
```

A simple vista, ya podemos observar algunos beneficios: **el código se reduce y simplifica muchísimo**, nos olvidamos de lidiar con el constructor, el valor de `this`, de bindear métodos que modifiquen el state, etc. 🎉🎉🎉

#### Agregando más estado al componente

A diferencia de cómo manejamos el state usando clases, donde definimos un único objeto `state` dentro del constructor que va a contener el estado del componente, con `useState` tenemos este mismo estado _modularizado_, por lo que si el componente necesita un state que contenga más de un valor, debemos llamar a `useState` las veces que sea necesario. Por ejemplo, si ahora queremos agregar un contador de clicks al botón, podemos hacer lo siguiente

```js
import React, { useState } from 'react';

function App() {
  const [text, setText] = useState('No clicks given.');
  const [counter, setCounter] = useState(0);

  return (
    <button onClick={() => {
      setText('button clicked');
      setCounter(counter => counter + 1);
    }}>
      {text} {counter ? `${counter} times.` : null}
    </button>
  );
}
```

> **Nota:** al igual que como sucede con `this.setState()` en un Class Component, cuando queremos actualizar estado basado en un estado anterior (como en el ejemplo de arriba), conviene pasarle un callback para que React pueda garantizarnos que el valor del state que utiliza es el actual, dado que son operaciones asincrónicas

En este caso, agregamos el state `counter`, inicializado en 0. Notemos también que `count` contiene el valor actual, por lo que no necesitamos acordarnos de [pasarle un callback para hacer referencia al valor previo del state](https://github.com/undefinedschool/notes-react-basics#modificando-el-state), como haríamos si estuviéramos usando `this.setState`).

#### Lazy State Initialization

Si el estado inicial de un valor del state depende de un cálculo pesado, podemos pasarle un callback como valor inicial a `useState`. Esto se conoce como [_lazy state initialization_](https://kentcdodds.com/blog/use-state-lazy-initialization-and-function-updates) y va a hacer que el cálculo se ejecute 1 vez, en el render inicial y no en cada re-render del componente.

```js
import React, { useState } from 'react';

function bigCompute() {
  // ...
}

function App() {
  const [count, setCount] = useState(() => bigCompute());

  // ...
}
```

### Component Lifecycle y side effects: [`useEffect`](https://reactjs.org/docs/hooks-effect.html)

Ok, y si ya no necesitamos clases, cómo reemplazamos los métodos de lifecycle de los componentes? La solución que proveen los hooks a esto es dejar de pensar en el lifecycle como lo hacíamos anteriormente (con la lógica del componente repartida entre diferentes métodos) y pasar a pensar en términos de **sincronización**: necesitamos sincronizar diferentes eventos/acciones que suceden fuera de la lógica de un componente de React (API request, manipulación del DOM) con algo interno del mismo (state).

Pensar en términos de _sincronización_ en lugar de eventos relacionados al _ciclo de vida_, nos permite agrupar fácilmente la lógica relacionada. Para manejar estos eventos (o [_side effects_](https://github.com/undefinedschool/notes-fp-js#side-effects)), utilizamos el hook [`useEffect`](https://reactjs.org/docs/hooks-effect.html).

`useEffect` nos permite manejar _side effects_ dentro de un componente. Recibe un callback como parámetro y, opcionalmente, un array, conocido como _dependency array_. El callback define la acción a ejecutar y los valores dentro del dependency array (opcional) definen cómo sincronizar estas acciones. 

#### Ejecutar `useEffect` si hay cambios en alguna dependencia (similar a `componentDidUpdate`)

```js
import React, { useState, useEffect } from 'react';

function App() {
  const [text, setText] = useState('0 clicks given.');

  useEffect(() => console.log(`Button text has changed to '${text}'.`), [text])

  return <button onClick={() => setText('clicked!')}>{text}</button>;
}
```

En el ejemplo de arriba, el callback va ejecutarse cada vez que el valor de `text` (que forma parte del state) cambie. Es por esto último que hablamos de un _array de dependencias_.

Si necesitáramos realizar un _fetch_ a una cierta API, cada vez que el valor `id` cambie, podríamos hacer algo como

```js
// ...
useEffect(() => {
  setLoader(true);
  
  fetchAPI(`BASE_URL/${id}`)
    .then(repos => {
      setRepos(repos);
      setLoader(false);
    })
    .catch(/* ... */)
}, [id]);
// ...
```

#### Ejecutar `useEffect` sólo en el 1er render (cuando el componente se monta)

`useEffect` nos va a permitir entonces simular las diferentes etapas del _lifecycle_ del componente. Por ejemplo, si quisiéramos ejecutar cierta acción cuando el componente termina de montarse por 1ra y única vez (como haríamos con el método `componentDidMount`), podemos definir el dependency array como un array vacío `[]`

```js
import React, { useState, useEffect } from 'react';

function App() {
  const [text, setText] = useState('0 clicks given.');

  useEffect(() => console.log('App component has been mounted.'), []);

  return <button onClick={() => setText('clicked!')}>{text}</button>;
}
```

De esta forma, el efecto (el callback que le pasamos a `useEffect`) va a ejecutarse una única vez.

#### Ejecutar `useEffect` en cada nuevo render

Y si queremos que el side effect se ejecute cada vez que el componente se vuelva a renderizar con _cualquier_ cambio en el state? En ese caso no le pasamos el dependency array (recordemos que este parámetro es opcional).

```js
import React, { useState, useEffect } from 'react';

function App() {
  const [text, setText] = useState('0 clicks given.');

  useEffect(() => console.log('App component has been re-rendered!'));

  return <button onClick={() => setText('clicked!')}>{text}</button>;
}
```

También **podemos definir múltiples side effects dentro de un componente**. Por ejemplo, si queremos leer/escribir un valor (como un contador) del state en `localStorage`, chequeando la 1ra vez si este valor ya existe y luego mantenerlo sincronizado cada vez que cambie, podemos hacer

```js
// ...
const [counter, setCounter] = useState(0);

useEffect(() => {
  const counter = localStorage.getItem('counter');

  if (counter) setCounter(Number.parseInt(counter));
}, []);

useEffect(() => {
  localStorage.setItem('counter', counter);
}, [counter]);
// ...
```

#### _cleanup_ (similar a `componentWillUnmount`)

El hook `useEffect` también provee la posibilidad de ejecutar una función de _cleanup_ (por ejemplo, para remover _event listeners_ o suscripciones a alguna API) después de ejecutarse, para lo cual debemos retornar una función al final del mismo. 

**Este hook va a ejecutarse justo antes de que el effect sea ejecutado en un nuevo render ó el componente sea removido del DOM**, evitando así los memory leaks.

```js
import React, { useState, useEffect };

import { subscribe, unsubscribe } from './api';

function Profile ({ username }) {
  const [profile, setProfile] = useState(null);

  useEffect(() => {
    subscribe(username, setProfile);

    return () => {
      unsubscribe(username);
      setProfile(null);
    }
  }, [username])

  if (!profile) {
    return <p>Loading...</p>;
  }

  return (
    <>
      <h1>@{profile.login}</h1>
      <img
        src={profile.avatar_url}
        alt={`Avatar for ${profile.login}`}
      />
      <p>{profile.bio}</p>
    </>
  );
}
```

### `useRef`

> _tl;dr_ nos permite preservar valores entre renders, sin triggerear un re-render del componente.

Este hook nos permite acceder directamente a un valor (ej: un nodo del DOM) directamente y mantener una referencia al mismo aún cuando el componente sea re-renderizado, durante su ciclo de vida. En cierta forma funciona como un _hack_ que rompe un poco el paradigma que nos plantea React, ya que estamos accediendo al DOM directamente.

[WIP]

### `useReducer`

> _tl;dr_ nos permite manejar data/state entre diferentes componentes.

[WIP]

### `useContext`

> _tl;dr_ para acceder a la `Context API` y manejar el state compartido entre diferentes componentes, evitando el _prop-drilling_

[WIP]

## Custom Hooks

Antes de que aparecieran los Hooks, la forma de compartir lógica de estado entre componentes eran utilizando [Higher-Order Components](https://github.com/undefinedschool/notes-react-advanced/#higher-order-components) y [Render Props](https://github.com/undefinedschool/notes-react-advanced/#render-props). Con la introducción de los Hooks, esto se simplifica bastante, ya que al ser los Hooks simplemente funciones, podemos extraer lógica que queremos reutilizar a funciones auxiliares, conocidas como _Custom Hooks_ y luego invocarlas desde los diferentes componentes.

### Ejemplo: `useFetch`

Supongamos que queremos evitar duplicar la lógica de _fetching_ de datos de una API, algo que necesitaríamos en distintos componentes. Para esto, vamos a crear el Custom Hook `useFetch` (tener en cuenta que esto es un ejemplo, se recomienda utilizar [`use-http`](https://use-http.com/) en lugar de crear nuestro propio hook).

Nuestro hook necesita mantener algo de estado: al menos `url` y `options` (para configurar el fetch) en el estado y funciones para setear las mismas (`setUrl`, `setOptions`). Además, necesitaremos cierta información sobre el request, si se encuentra pendiente, ya finalizó u ocurrió algún error, para actualizar nuestra UI en consecuencia.

- data (la data que retorna el request)
- error (si el request falla por alguna razón)
- loading (un booleano que indica si el request finalizó y ya tenemos la respuesta)

Además, vamos a querer ejecutar un request cada vez que `url` u `options` cambien. Para esto, podemos utilizar `useEffect`.

```js
import { useEffect, useState } from 'react';

function useFetch(url, options) {
  const [data, setData] = useState('');
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(
    () =>
      fetch(url, options)
        .then(response => response.json())
        .then(data => setData(data))
        .catch(e => setError(e))
        .finally(() => setLoading(false)),
    [url, options]
  );

  return [data, loading, error];
}

export default useFetch;
```
