# Mise en place

## Déclaration des reducers

Dans le fichier app/Resources/js/app.js, ajouter ces lignes en début de fichier :

```js
import { createStore, combineReducers } from 'redux';
import { reducer as formReducer } from 'redux-form';
import reducer from './reducer';
import { Provider } from 'react-redux';
import { StudentsList } from './containers';
```

[Comme l'explique la documentation de Redux](http://redux.js.org/docs/basics/Reducers.html), nous avons besoin de déclarer des actions \(ajouter ceci, modifier cela\) avec des paramètres mais sans modifier l'état de l'application.

Le ou les reducers se chargeront de modifier l'état \(ce que nous verrons plus tard\).

La déclaration des reducers dans le fichier app.js se fait de la façon suivante :

```js
const reducers = {
  reducer,
  form: formReducer,
}
const reduc = combineReducers(reducers);
const store = createStore(reduc);
```

La ligne `form: formReducer,`provient de Redux-Form, une librairie développée sur la philsophie Redux, et spécialisée dans la validation et le traitement de formulaires HTML, ce que l'on détaillera plus tard. La ligne juste au-dessus est le reducer que l'on écrira pour notre application de gestion d'élèves.

Les deux dernières lignes sont nécessaires pour que Redux fonctionne : avec plusieurs reducers, il est nécessaire de les combiner. Ensuite, quelque soit le nombre de reducers, il faut créer un store, ce qui va conserver l'état des objets au sein de l'application \(tant que le navigateur est ouvert en tout cas, à moins de gérer la persistance mais cela va au-delà de notre sujet\).

## Mise en place du store

Enfin, Redux demande d'empaqueter toute l'appli dans un composant de haut niveau \(High Order Component, ou HOC - un article à retenir pour bien comprendre : [http://putaindecode.io/fr/articles/js/react/higher-order-component/](http://putaindecode.io/fr/articles/js/react/higher-order-component/)\)

Cela se passe à la manière de ce que demandait React-Router, qui agissait en tant que wrapper sur toutes les routes de l'application. Sans React-Router, cela donnerait :

```js
<Provider store={store}>
    <StudentsList />
</Provider>
```

Mais avec le code existant, voici ce qu'il faut pour notre fichier \(au passage, j'ai ajouté la route vers la page de gestion de classe numérique que nous allons construire dans ce tuto\) :

```js
<Provider store={store}>
    <Router history={browserHistory}>
        <Route path="/" component={Layout}>
            <IndexRoute component={Home} />
            <Route path="/home" component={Home} />
            <Route path="/page1" component={Page1} />
            <Route path="/page2" component={Page2} />
            <Route path="/students" component={StudentsList} />
            <Route path="*" component={NoMatch} />
        </Route>
    </Router>
  </Provider>
```

Finalement, voici le fichier app.js :

```js
import React from 'react';
import ReactDom from 'react-dom';
import { Router, Route, IndexRoute, browserHistory } from 'react-router';

import { createStore, combineReducers } from 'redux';
import { reducer as formReducer } from 'redux-form';
import reducer from './reducer';
import { Provider } from 'react-redux';

import Layout from './Layout';
import Home from './components/Home';
import Page1 from './components/Page1';
import Page2 from './components/Page2';
import NoMatch from './components/NoMatch';
import StudentsContainer from './container';

const reducers = {
  reducer,
  form: formReducer,
}
const reduc = combineReducers(reducers);
const store = createStore(reduc);

ReactDom.render((
  <Provider store={store}>
    <Router history={browserHistory}>
        <Route path="/" component={Layout}>
            <IndexRoute component={Home} />
            <Route path="/home" component={Home} />
            <Route path="/page1" component={Page1} />
            <Route path="/page2" component={Page2} />
            <Route path="/students" component={StudentsContainer} />
            <Route path="*" component={NoMatch} />
        </Route>
    </Router>
  </Provider>
), document.getElementById('app'));
```

Ca commence à faire fouillis. Pour être plus propre, on pourrait mettre la déclaration des reducers dans un sous-fichier, et même gérer les routes ailleurs. Mais pour ce tutoriel, on va rester avec cette structure de fichiers simple.

La base de Redux est en place : des reducers \(contenant des fonctions pures modifiant l'état de l'application\) sont déclarés pour être stockés dans le store. Ce store est ensuite utilisé dans un HOC, le provider, qui wrappe tout ce qu'il se passe dans l'application. Toute l'application ? Non, un élément irréductible résiste encore et toujours à Redux, c'est l'état des routes. Mais ce n'est pas grave, on pourrait mettre le package React-Router-Redux si on le voulait vraiment, permettant à Redux d'enregistrer les routes. Mais ce ne sera pas le cas ici pour simplifier l'exemple.

