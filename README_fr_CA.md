<img src='https://redux-saga.js.org/logo/0800/Redux-Saga-Logo-Landscape.png' alt='Redux Logo Landscape' width='800px'>

# redux-saga

[![npm version](https://img.shields.io/npm/v/redux-saga.svg)](https://www.npmjs.com/package/redux-saga)
[![CDNJS](https://img.shields.io/cdnjs/v/redux-saga.svg)](https://cdnjs.com/libraries/redux-saga)
[![npm](https://img.shields.io/npm/dm/redux-saga.svg)](https://www.npmjs.com/package/redux-saga)
[![Build Status](https://travis-ci.org/redux-saga/redux-saga.svg?branch=master)](https://travis-ci.org/redux-saga/redux-saga)
[![Join the chat at https://gitter.im/yelouafi/redux-saga](https://badges.gitter.im/yelouafi/redux-saga.svg)](https://gitter.im/yelouafi/redux-saga?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![OpenCollective](https://opencollective.com/redux-saga/backers/badge.svg)](#backers)
[![OpenCollective](https://opencollective.com/redux-saga/sponsors/badge.svg)](#sponsors)

`redux-saga` is a library that aims to make application side effects (i.e. asynchronous things like data fetching and impure things like accessing the browser cache) easier to manage, more efficient to execute, simple to test, and better at handling failures.

The mental model is that a saga is like a separate thread in your application that's solely responsible for side effects. `redux-saga` is a redux middleware, which means this thread can be started, paused and cancelled from the main application with normal redux actions, it has access to the full redux application state and it can dispatch redux actions as well.

It uses an ES6 feature called Generators to make those asynchronous flows easy to read, write and test. *(if you're not familiar with them [here are some introductory links](https://redux-saga.js.org/docs/ExternalResources.html))* By doing so, these asynchronous flows look like your standard synchronous JavaScript code. (kind of like `async`/`await`, but generators have a few more awesome features we need)

You might've used `redux-thunk` before to handle your data fetching. Contrary to redux thunk, you don't end up in callback hell, you can test your asynchronous flows easily and your actions stay pure.


## translation 

Redux-saga est une bibliothèque qui vise à rendre les effets secondaires de l'application (c'est-à-dire des choses asynchrones comme l'extraction de données et des objets impurs comme l'accès au cache du navigateur) plus faciles à gérer, plus efficaces à exécuter, simples à tester et mieux à gérer les pannes.

Le modèle mental est qu'une saga est comme un fil distinct dans votre application qui est seul responsable des effets secondaires. Redux-saga est un middleware redux, ce qui signifie que ce thread peut être démarré, arrêté et annulé à partir de l'application principale avec des actions redux normales, il a accès à l'état complet de l'application redux et il peut également envoyer des actions redux.

Il utilise une fonction ES6 appelée Génératrices pour rendre ces flux asynchrones faciles à lire, à écrire et à tester. (Si vous ne les connaissez pas, vous trouverez ici des liens d'introduction). Ces flux asynchrones ressemblent à votre code JavaScript synchrone standard. (Genre d'async / await, mais les générateurs ont quelques autres fonctionnalités incroyables dont nous avons besoin)

Vous avez probablement utilisé redux-thunk avant de gérer votre recherche de données. Contrairement à redux thunk, vous ne finissez pas en rappel, vous pouvez tester vos flux asynchrones facilement et vos actions restent pure.


# Getting started
# Commencer

## Install
## Installer

```sh
$ npm install --save redux-saga
```
or

```sh
$ yarn add redux-saga
```

Alternatively, you may use the provided UMD builds directly in the `<script>` tag of an HTML page. See [this section](#using-umd-build-in-the-browser).

Alternativement, vous pouvez utiliser les compilations UMD fournies directement dans la balise `<script>` d'une page HTML. Voir cette section.

## Usage Example

## Exemple d'utilisation

Suppose we have a UI to fetch some user data from a remote server when a button is clicked. (For brevity, we'll just show the action triggering code.)


Supposons que nous ayons une interface utilisateur pour récupérer des données utilisateur à partir d'un serveur distant lorsqu'un bouton est cliqué. (Pour plus de précision, nous allons simplement montrer le code déclencheur d'action.)

```javascript
class UserComponent extends React.Component {
  ...
  onSomeButtonClicked() {
    const { userId, dispatch } = this.props
    dispatch({type: 'USER_FETCH_REQUESTED', payload: {userId}})
  }
  ...
}
```

The Component dispatches a plain Object action to the Store. We'll create a Saga that watches for all `USER_FETCH_REQUESTED` actions and triggers an API call to fetch the user data.



Le Composant expédie une action Objet simple à la boutique. Nous allons créer une Saga qui surveille toutes les actions `USER_FETCH_REQUESTED` et déclenche un appel API pour récupérer les données utilisateur.

#### `sagas.js`

```javascript
import { call, put, takeEvery, takeLatest } from 'redux-saga/effects'
import Api from '...'

// worker Saga: will be fired on USER_FETCH_REQUESTED actions
function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}

/*
  Starts fetchUser on each dispatched `USER_FETCH_REQUESTED` action.
  Allows concurrent fetches of user.

  Démarre fetchUser sur chaque action envoyée 'USER_FETCH_REQUESTED`.
  Permet des recherches simultanées de l'utilisateur.
*/
function* mySaga() {
  yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

/*
  Alternatively you may use takeLatest.

  Does not allow concurrent fetches of user. If "USER_FETCH_REQUESTED" gets
  dispatched while a fetch is already pending, that pending fetch is cancelled
  and only the latest one will be run.


  Vous pouvez également utiliser takeLatest.

  Ne permet pas les accès concurrents de l'utilisateur. Si "USER_FETCH_REQUESTED" obtient
  Expédié alors qu'une tentative est déjà en attente, que l'extraction en attente est annulée
  Et seul le dernier sera exécuté.
*/
function* mySaga() {
  yield takeLatest("USER_FETCH_REQUESTED", fetchUser);
}

export default mySaga;
```

To run our Saga, we'll have to connect it to the Redux Store using the `redux-saga` middleware.


Pour exécuter notre Saga, nous devrons le connecter au Redux Store en utilisant le middleware `redux-saga`.

#### `main.js`

```javascript
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import reducer from './reducers'
import mySaga from './sagas'

// create the saga middleware
// Créer le middleware de la saga
const sagaMiddleware = createSagaMiddleware()
// mount it on the Store
// Montez-le sur le magasin
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)

// then run the saga
// Puis exécutez la saga
sagaMiddleware.run(mySaga)

// render the application
// Rendre l'application
```

# Documentation

- [Introduction](https://redux-saga.js.org/docs/introduction/BeginnerTutorial.html)
- [Basic Concepts](https://redux-saga.js.org/docs/basics/index.html)
- [Advanced Concepts](https://redux-saga.js.org/docs/advanced/index.html)
- [Recipes](https://redux-saga.js.org/docs/recipes/index.html)
- [External Resources](https://redux-saga.js.org/docs/ExternalResources.html)
- [Troubleshooting](https://redux-saga.js.org/docs/Troubleshooting.html)
- [Glossary](https://redux-saga.js.org/docs/Glossary.html)
- [API Reference](https://redux-saga.js.org/docs/api/index.html)

# Translation

- [Chinese](https://github.com/superRaytin/redux-saga-in-chinese)
- [Traditional Chinese](https://github.com/neighborhood999/redux-saga)
- [Japanese](https://github.com/redux-saga/redux-saga/blob/master/README_ja.md)
- [Korean](https://github.com/mskims/redux-saga-in-korean)
- [Russian](https://github.com/redux-saga/redux-saga/blob/master/README_ru.md)

# Using umd build in the browser
# Utilisation de umd build dans le navigateur

There is also a **umd** build of `redux-saga` available in the `dist/` folder. When using the umd build `redux-saga` is available as `ReduxSaga` in the window object. This enables you to create Saga middleware without using ES6 `import` syntax like this:


Il existe également une version ** umd ** de `redux-saga` disponible dans le dossier` dist / `. Lors de l'utilisation de la «redux-saga» de la compilation umd est disponible en tant que «ReduxSaga» dans l'objet fenêtre. Cela vous permet de créer un middleware Saga sans utiliser la syntaxe ES6 `import` comme ceci:

```javascript
var sagaMiddleware = ReduxSaga.default()
```

The umd version is useful if you don't use Webpack or Browserify. You can access it directly from [unpkg](https://unpkg.com/).

The following builds are available:



La version μd est utile si vous n'utilisez pas Webpack ou Browserify. Vous pouvez y accéder directement à partir de [unpkg](https://unpkg.com/).

Les versions suivantes sont disponibles:


- [https://unpkg.com/redux-saga/dist/redux-saga.js](https://unpkg.com/redux-saga/dist/redux-saga.js)
- [https://unpkg.com/redux-saga/dist/redux-saga.min.js](https://unpkg.com/redux-saga/dist/redux-saga.min.js)

**Important!** If the browser you are targeting doesn't support *ES2015 generators*, you must provide a valid polyfill, such as [the one provided by `babel`](https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.25/browser-polyfill.min.js). The polyfill must be imported before **redux-saga**:


Important! Si le navigateur que vous ciblez ne prend pas en charge les générateurs ES2015, vous devez fournir un polyfill valide, tel que celui fourni par babel. Le polyfill doit être importé avant redux-saga:

```javascript
import 'babel-polyfill'
// then
import sagaMiddleware from 'redux-saga'
```

# Building examples from sources
# Construire des exemples à partir de sources

```sh
$ git clone https://github.com/redux-saga/redux-saga.git
$ cd redux-saga
$ npm install
$ npm test
```

Below are the examples ported (so far) from the Redux repos.

### Counter examples

There are three counter examples.

#### counter-vanilla

Demo using vanilla JavaScript and UMD builds. All source is inlined in `index.html`.

To launch the example, just open `index.html` in your browser.


Voici les exemples portés (à ce jour) dans les remises de Redux.

### Counter examples

Il y a trois contre-exemples.

#### counter-vanilla

Démonstration en utilisant les compilations vanité JavaScript et UMD. Toute source est intégrée dans `index.html`.

Pour lancer l'exemple, ouvrez `index.html` dans votre navigateur.


> Important: your browser must support Generators. Latest versions of Chrome/Firefox/Edge are suitable.

Important: votre navigateur doit prendre en charge les générateurs. Les dernières versions de Chrome / Firefox / Edge conviennent.

#### counter

Demo using `webpack` and high-level API `takeEvery`.

```sh
$ npm run counter

# test sample for the generator
$ npm run test-counter
```

#### cancellable-counter

Demo using low-level API to demonstrate task cancellation.

```sh
$ npm run cancellable-counter
```

### Shopping Cart example

```sh
$ npm run shop

# test sample for the generator
$ npm run test-shop
```

### async example

```sh
$ npm run async

# test sample for the generators
$ npm run test-async
```

### real-world example (with webpack hot reloading)

```sh
$ npm run real-world

# sorry, no tests yet
```

### TypeScript

Redux-Saga with TypeScript requires `DOM.Iterable` or `ES2015.Iterable`. If your `target` is `ES6`, you are likely already set, however, for `ES5`, you will need to add it yourself.
Check your `tsconfig.json` file, and the official <a href="https://www.typescriptlang.org/docs/handbook/compiler-options.html">compiler options</a> documentation.

Redux-Saga avec TypeScript nécessite DOM.Iterable ou ES2015.Iterable. Si votre cible est ES6, vous êtes probablement déjà configuré pour ES5, vous devrez l'ajouter vous-même. Vérifiez votre fichier tsconfig.json et la documentation officielle des options du compilateur.

### Logo

You can find the official Redux-Saga logo with different flavors in the [logo directory](logo).


### Video
https://www.youtube.com/watch?v=ByQknrU42mY
https://www.youtube.com/watch?v=o3A9EvMspig
https://www.youtube.com/watch?v=hfrnlxIZm3E
https://www.youtube.com/watch?v=4aDyN5cOFBw
https://www.youtube.com/watch?v=EifOGwAW5ZM
https://www.youtube.com/watch?v=dOqhOg4ekRk

### French helper for the translation

facilite ainsi l’utilisation de bibliothèques comme redux-saga, où l’on attend un support pour les générateurs.
https://www.monde-informatique.com/typescript-2-3-est-plus-intelligent-sur-les-normes-javascript/
https://www.occitech.fr/blog/2016/03/comment-apprehender-lecosysteme-react/
http://joelhooks.com/blog/2016/03/20/build-an-image-gallery-using-redux-saga
### Backers
Support us with a monthly donation and help us continue our activities. \[[Become a backer](https://opencollective.com/redux-saga#backer)\]

<a href="https://opencollective.com/redux-saga/backer/0/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/0/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/1/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/1/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/2/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/2/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/3/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/3/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/4/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/4/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/5/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/5/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/6/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/6/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/7/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/7/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/8/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/8/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/9/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/9/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/10/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/10/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/11/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/11/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/12/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/12/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/13/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/13/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/14/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/14/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/15/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/15/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/16/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/16/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/17/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/17/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/18/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/18/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/19/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/19/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/20/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/20/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/21/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/21/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/22/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/22/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/23/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/23/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/24/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/24/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/25/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/25/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/26/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/26/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/27/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/27/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/28/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/28/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/29/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/29/avatar.svg"></a>


### Sponsors
Become a sponsor and get your logo on our README on Github with a link to your site. \[[Become a sponsor](https://opencollective.com/redux-saga#sponsor)\]

<a href="https://opencollective.com/redux-saga/sponsor/0/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/0/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/1/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/1/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/2/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/2/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/3/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/3/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/4/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/4/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/5/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/5/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/6/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/6/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/7/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/7/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/8/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/8/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/9/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/9/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/10/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/10/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/11/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/11/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/12/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/12/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/13/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/13/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/14/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/14/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/15/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/15/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/16/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/16/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/17/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/17/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/18/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/18/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/19/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/19/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/20/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/20/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/21/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/21/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/22/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/22/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/23/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/23/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/24/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/24/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/25/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/25/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/26/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/26/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/27/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/27/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/28/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/28/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/29/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/29/avatar.svg"></a>
