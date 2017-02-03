# Pour aller plus loin

Cet exemple, très basique en termes de fonctionnalités, bien que complexe dans l'architecture, doit pouvoir être complété pour de vraies applications. Notamment, le fait de contacter une API pour recevoir et envoyer de la donnée.

Pour cela, il faut un middleware et Redux fournit un mécanisme pour s'en servir à travers applyMiddleware. Un middleware bien utilisé dans la communauté est Redux-Thunk, spécialisé dans la communication avec des API.

Pour s'en servir, il faudrait modifier app.js de la manière suivante \(et installer le paquet redux-thunk au préalable\) :

```js
import { ..., applyMiddleware } from 'redux';
import thunk from 'redux-thunk';

...
const store = createStore(reduc, applyMiddleware(thunk));
```

Redux-thunk est donc intégré au store. On peut modifier une action existante dans le fichier actions.js. Par exemple, l'action d'affichage du formulaire se faisait comme ceci :

```js
export function toggleForm(id) {
  return {
    type: 'TOGGLE_FORM',
    id: id
  };
}
```

Cette action pourrait avoir à aller chercher les informations de l'élève non plus dans un fichier de fixture mais sur le réseau à travers une API de cette manière :

```js
export function toggleForm(id) {  
  return dispatch =>
    fetch(`/api/student/${id}`, {
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ id: id }),
    })
    .then(res => res.json())
    .then(data => {
      return dispatch({
        type: 'TOGGLE_FORM',
        payload: data,
      });
    })
    .catch(err => console.log('err: ', err));
  }
```

On utilise ici la fonction dispatch de Redux pour enchainer les événements. Le premier étant l'appel à l'API. Une fois récupérées les données correspondants à un identifiant d'élève, on peut dispatcher l'événement TOGGLE\_FORM auquel Redux saura répondre à travers le reducer \(il faudra aussi penser à modifier le reducer pour réagir à la nouvelle donnée reçue\).

Avec cette structure, désormais, le store embarque un \(ou plusieurs\) middleware pour permettre aux créateurs d'actions d'intégrer des fonctions impures \(side effect tel qu'un appel à une ressource externe à l'application\). Le reducer reste pur quant à lui.

