# Using Saga Helpers

`redux-saga` ofrece algunas functiones auxiliares _(helpers)_ que ocultan funciones internas que ejecutan tareas _(tasks)_ cuando acciones son enviadas al almacén de datos _(Store)_.

Las funciones auxiliares están hechas sobre el API de bajo nivel de `redux-saga`. En la seccion avanzada, se analizará como esas funciones puedes ser implementadas.

La primera function `takeEvery` _(tomarTodas)_ es la más común y provee un comportamiento similar al que se tiene en `redux-thunk`.

Ejemplificando esto con un caso de común de AJAX. En cada clic en un boton que pide datos al servidor se envía una accion de tipo `FETCH_REQUESTED`. Se quiere interceptar esta acción y ejecutar una tarea _(task)_ que va a pedir datos del servidor.

Primero creamos esa tarea que ejecutrá la acción asíncrona:

```javascript
import { call, put } from 'redux-saga/effects'

export function* fetchData(action) {
   try {
      const data = yield call(Api.fetchUser, action.payload.url)
      yield put({type: "FETCH_SUCCEEDED", data})
   } catch (error) {
      yield put({type: "FETCH_FAILED", error})
   }
}
```

Para interceptar cada acción `FETCH_REQUESTED` enviada y ejecutar la tarea:

```javascript
import { takeEvery } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeEvery('FETCH_REQUESTED', fetchData)
}
```
En el caso anterior `takeEvery` permite que multiples ejecuciones de la tarea `fetchData` puedan ser iniciadas de manera concurrente. En cualquier momento, se puede iniciar una nueva tarea `fetchData` mientras que hay una o más tareas `fetchData` que aún no han terminado de ejecutarse.

Si se desea obtener la última respuesta del servidor de una serie de tareas iniciadas (esto es, siempre obtener los datos mas recientes) podemos usar la función auxiliar _(helper)_ `takeLatest`:

```javascript
import { takeLatest } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeLatest('FETCH_REQUESTED', fetchData)
}
```

A diferencia de `takeEvery`, `takeLatest` solo permite que una tarea `fetchData` se ejecute en el mismo lapso de tiempo, siendo esta la última en ser ejecutada. Si una tarea es fue ejecutada con anterioridad y aún continua ejecutandose cuando una nueva tarea `fetchData` es ejecutada la tarea `fetchData` anterior será automaticamente cancelada y no terminará su ejecución.

Si se tienen multiples Sagas observando por diferentes tipos de acciones, se pueden crear multiples _watchers (observadores)_ con functiones auxiliares que se comportaran como si se hubiese usado la función `fork` para lanzar esos observadores (hablaremos de la función `fork` después, por ahora hay que considerarla como si fuese un Efecto que permite tener multiples sagas en _background_).

Por ejemplo:

```javascript
import { takeEvery } from 'redux-saga/effects'

// FETCH_USERS
function* fetchUsers(action) { ... }

// CREATE_USER
function* createUser(action) { ... }

// Las ejecuta en paralelo
export default function* rootSaga() {
  yield takeEvery('FETCH_USERS', fetchUsers)
  yield takeEvery('CREATE_USER', createUser)
}
```
