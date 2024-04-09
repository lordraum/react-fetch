---
title: 'React fetch'
tutorial: 'Cómo Consumir APIs en REACT como un PROFESIONAL'
tutor: 'Carlos Azaustre'
---

[enlace tutorial](https://www.youtube.com/watch?v=6u1RHUoXIPI&t)

# Fetch en React

Es necesario cargar los hooks useState y useEffect. El hook useEffect es necesario para hacer la petición HTTP.

*Recordatorio*
  - La dependencia [] array vacío para useEffect significa que solo se ejecutará una vez cuando se monte el componente.

En el useEffect se setea la data como estado, y de esta forma poder utilizarlo para renderizar contenido de la data.

```jsx
import { useState, useEffect } from 'react'

const App = () => {
  const [data, setData] = useState(null)

  useEffect(() => {
    fetch('https://jsonplaceholder.typicode.com/users')
      .then(res => res.json())
      .then(dataJSON => setData(dataJSON))
  }, [])

  return (
    <div>
      <h1>Fetch like a pro</h1>
      <ul>
        {
          data?.map(({ id, name }) => (
            <li key={id}>{id} - {name}</li>
          ))
        }
      </ul>
    </div>
  )
}

export default App
```

## useFetch

Se puede crear un custom hook `useFetch` para facilitar la reutilización de `fetch`. Utilizar la url como input. la data a devolver es aconsejable usarla como objeto porque es más fácil de desustructurar.

## Loading

Para gestionar qué hacer mientras la data está cargando, se crea otro estado. Cuyo valor inicial será true, y cuando se haga el fetch, es decir cuando se haya cargado la data será false. La mejor forma de establecer el nuevo estado de loading es con el método `finally`, que se ejecuta hasta que se resuelven todas las promesas. Finalmente devolver loading, junto a data.

Cargar loading en el componente donde se ejecuta el fetch con un renderizado condicional.

```jsx
import { useState, useEffect } from 'react'

export const useFetch = (url) => {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(dataJSON => setData(dataJSON))
      .finally(() => setLoading(false))
  // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [])

  return { data, loading }
}
```

```jsx
<ul>
  {loading && <li>Loading ...</li>}
  {
    data?.map(({ id, name }) => (
      <li key={id}>{id}- {name}</li>
    ))
  }
</ul>
```
Para hacer más fácil la comprobación en las herramientas de desarrollador activar --> `network/network conditions/slow 3G`

## Manejo de errores
  - Crear estado `error` con valor por inicial `null`
  - Se utiliza el método de las promesas `catch` para establecer `error`
  - Devolver `error` en el objeto que retorna la función
  - Cargar `error` en el componente

```jsx
export const useFetch = (url) => {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(x => setError('Ocurrió un error')) // Simulando un error
      .catch(error => setError(error))
      .finally(() => setLoading(false))
  }, [])

  return { data, loading, error }
}
```

```jsx
const { data, loading, error } = useFetch(URL)
//...
{error && <li>Error: {error}</li>}
```

## Abort controller

Utilidad de la plataforma que permite cancelar peticiones.

Declarar al inicio del useEffect --> `const abortController = new AbortController()`

Esto permite rastrear la petición, para después devolver --> `return () => abortController.abort()`

Es importante recordar que las funciones que se retornan en un useEffect se utilizarán cuando se desmonte el componente. A estas funciones se les conoce como `función de limpieza`

### handleFunction

Se puede combrar como --> `handleCancelRequest()`

Función que encapsulará a `abortController.abort()`, esto para exportarla en el fetch y poder hacer uso de esta en los componentes. Es necesario para usarla crear un estado --> `controller`, con valor `null`, que se seteará con el valor `abortController` en el effect.

## Render As You Fetch

El componente `suspense` permite un mnejo más eficiente del loading. La petición a la API, se realiza por fuera del componente y sin utilizar un custom hook.

Crear archivo js para hacer la petición API --> `fetchData.js`

Crear función que encapsulará el fetch. Este fecth se hará sin async await.

Se delvolverá una función --> `getSuspender()`, esta será la que se encargue de la asincronía.

### getSuspender()

Mediante casos de estado --> `pending`, `success`, `error` --> Resuelve la solicitud (promesa)

### Suspense

Componente que renderiza algo mientras los datos cargan.

Envuelve a los componenetes donde queremos utilizarlo

`fallback` --> el jsx de lo que deseamos cargar

```javascript
const getSuspender = (promise) => {
  let status = 'pending'
  let response

  const suspender = promise.then(
    (res) => {
      status = 'success'
      response = res
    },
    (err) => {
      status = 'error'
      response = err
    }
  )

  const read = () => {
    switch (status) {
      case 'pending': throw suspender
      case 'error': throw response
      default: return response
    }
  }

  return { read }
}

export function fetchData (url) {
  const promise = fetch(url)
    .then((response) => response.json())
    .then((json) => json)

  return getSuspender(promise)
}

// App.jsx

import { fetchData } from './fetchData'
import './App.css'
import { Suspense } from 'react'

function App () {
  const data = apiData.read()
  // etc
}
```