# La mécanique \(ou comment tout s'emboîte\)

Tous ces fichiers et ces ajouts peuvent paraître confus. Pour résumer, nous avons enrichi le fichier d'entrée de l'application \(app.js\) pour qu'il prenne en compte Redux. Ce qu'il faut retenir c'est que le \(ou les\) reducer\(s\) est ce qui contiendra les fonctions qui manipuleront l'état de l'application. C'est donc le coeur de la mécanique de Redux.

Ces reducers sont combinés dans un store, qui est lui-même embarqué dans un HOC \(High Order Component\), c'est-à-dire un composant qui wrappe l'ensemble de l'application pour en gérer l'état.

## Reducer

Passons donc à ce fameux reducer. Pour simplifier notre exemple, nous allons créer des fixtures, qui seront gérées en mémoire : chaque rechargement de page ou réouverture du navigateur réinitilisera l'état de l'application. Pour rappel, nous gérons une classe d'élèves. Créons des élèves factices dans fixtures.js au même niveau que app.js[^1] :

```js
import Immutable, { List, Map } from 'immutable';

export const students = List([
  Map({ id: 1465171631163, firstName: 'Jean', lastName: 'Borotra', level: '6e', average: '10', isVisible: false, isUpdated: false }),
  Map({ id: 1465171646328, firstName: 'Jacques', lastName: 'Brugnon', level: '5e', average: '19', isVisible: false, isUpdated: false }),
  Map({ id: 1465171723584, firstName: 'Henri', lastName: 'Cochet', level: '3e', average: '14', isVisible: false, isUpdated: false }),
  Map({ id: 1465171659936, firstName: 'René', lastName: 'Lacoste', level: '6e', average: '8', isVisible: false, isUpdated: false }),
  Map({ id: 1465171707368, firstName: 'Suzanne', lastName: 'Lenglen', level: '6e', average: '15', isVisible: false, isUpdated: false }),
]);
```

On utilise ici la bibliothèque Immutable.js pour respecter la philosophie Redux qui ne modifie pas un enregistrement à chaque fois mais le recrée, permettant des comparaisons très rapides entre objets puisqu'elle se fera sur la référence de l'objet plutôt que sur l'objet lui-même. Cela devient intéressant sur de gros objets avec plusieurs niveaux d'imbrication. Ce n'est donc pas obligatoire pour notre simple application. Mais c'est un concept important à connaître.

Chaque élève a donc un identifiant, un prénom, un nom, un niveau de classe \(6e, 5e...\), une moyenne, une propriété permettant de savoir s'il est visible ou non et une dernière propriété pour savoir s'il a été mis à jour ou non. Ce sont ces informations qui seront gérées dans le formulaire.

Le principe du reducer est de réagir à des actions pour produire un nouvel état dans l'application. Ce seront donc des fonctions pures, sans effet de bord \(side effect, par exemple des appels à une API externe\) et qui vont aussi utiliser l'immutabilité pour recréer un état plutôt que de modifier celui existant. D'où l'intérêt d'utiliser Immutable.js \(même si non obligatoire comme précisé plus haut\).

Voici notre reducer \(à placer dans un fichier reducer.js au même niveau que app.js\) :

```js
import Immutable, { List, Map } from 'immutable';
import { students } from './fixtures';

export default function(state = students, action) {
  switch(action.type) {
      case 'TOGGLE_FORM':
          return state.map(item => {  
            if(item.get('id') === action.id) {
              return item.set('isVisible', !item.get('isVisible')).set('isUpdated', false);
            } else {
              return item;
            }
          });
      case 'UPDATE_FORM':
          return state.map(item => {
            if(item.get('id') === action.id) {
              return Immutable.fromJS(action.item).set('isUpdated', true);
            } else {
              return item;
            }
          }); 
    case 'ADD_STUDENT': 
          action.item.id = action.id;
          return state.push(Map(action.item).set('isVisible', false).set('isUpdated', false));
    case 'DELETE_STUDENT': 
          return state.filter(function(item) {
              return item.get("id") != action.id;
          });
    default:
      return state;
  }
}
```

Ici, nous gérons 5 cas :

* TOGGLE\_FORM : l'affichage ou le masquage du détail d'un élève
* UPATE\_FORM : la mise à jour des informations d'un élève après soumission du formulaire Redux-Form. Remarquez l'utilisation d'Immutable pour modifier l'objet contenant l'élève : Immutable va se charger lui-même de recréer un objet.
* ADD\_STUDENT : la création d'un nouvel élève après soumission du formulaire Redux-Form \(le même que la modification\).
* DELETE\_STUDENT : la suppression d'un élève par création d'un objet contenant les élèves sauf celui à supprimer. Cela respecte le principe d'immutabilité.
* Défaut : si aucune action ne correspond à la demande, le renvoi de l'état actuel \(pour affichage du tableau global des élèves en fait\).

Ce reducer, simple en soi, reçoit donc l'état actuel des élèves \(qui est les fixtures précédemment définies au démarrage de l'application\) et une action. Comment cette action est-elle déclenchée ? C'est ce que gère un fichier qu'on appelle "action creators", créateur d'actions.

## Actions creator

Créons un fichier actions.js au même niveau que app.js :

```js
// succinct hack for generating passable unique ids
const uid = () => new Date().valueOf();

export function addStudent(item) {
  return {
    type: 'ADD_STUDENT',
    item: item,
    id: uid()
  };
}

export function deleteStudent(id) {
  return {
    type: 'DELETE_STUDENT',
    id: id
  };
}

export function toggleForm(id) {
  return {
    type: 'TOGGLE_FORM',
    id: id
  };
}

export function updateForm(item) {
  return {
    type: 'UPDATE_FORM',
    item: item,
    id: item.id,
  };
}
```

On retrouve les 4 cas \(sauf le défaut donc\) vus dans le reducer. Rien de bien complexe ici : pour chaque fonction, on crée un objet qui contient un type \(obligatoire\), et une charge utile \(payload en english\) qui sera un ou plusieurs paramètres à appliquer un objet \(un élève par exemple\). Ce fichier est presque trop simple. Pourquoi créer un fichier qui ne fait que répartir des actions au reducer ? On pourrait directement appeler les fonctions du reducer plutôt que de passer par ce fichier intermédiaire...

Dans la logique Redux, ce fichier d'actions est très important, parce qu'il constitue la seule source de vérité \(single source of truth\) pour le store, c'est-à-dire que le store ne connait que les actions à appeler, pas comment elles doivent être prises en charge. Cela permet un découplage des responsablilités. Un autre avantage est de déléguer à ce fichier créateur d'actions des actions asynchrones \(le côté "impur", appelé aussi side effect\). Ainsi, le reducer ne conserve que des fonctions pures, limitant les effets de bord, et facilitant les tests automatiques \(puisque qu'une fonction pure ne fait qu'une et une seule chose, alors qu'un action creator peut avoir à faire tout un tas d'opérations de validation et de chargement de données asynchrones en amont du reducer\).

Pour notre application, le créateur d'actions sera très léger, mais cela ne signifie pas qu'on peut le fusionner avec le reducer pour gagner quelques lignes de code.

## Container

Le container est le dernier aspect pour lier les composants et Redux. Le composant StudentsList étant le parent des autres composants de gestion des élèves \(Student et StudentForm\), c'est lui qui sera lié à Redux. Dans ce cas, Student est un composant de présentation, c'est-à-dire qu'il ne fera qu'afficher des informations transmises par un conteneur. Ce conteneur, sous-entendu conteneur de composant de présentation, sera StudentsList, le seul à être lié directement à Redux.

Pour cela, on peut ajouter en bas du fichier StudentsList.js une partie "container" issue de Redux. Dans un souci de lisibilité, on peut aussi extraire la partie container pour la mettre dans un fichier à part. Créons le fichier container.js au même niveau que app.js :

```js
import { connect } from 'react-redux';
import StudentsList from './components/StudentsList';
import { toggleForm, updateForm, addStudent, deleteStudent } from './actions';

const StudentsContainer = connect(
  function mapStateToProps(state) {
    return { students: state };
  },
  function mapDispatchToProps(dispatch) {
    return {
        toggleForm: id      => dispatch(toggleForm(id)),
        updateForm: data    => dispatch(updateForm(data)),
        addStudent: data    => dispatch(addStudent(data)),
        deleteStudent: data => dispatch(deleteStudent(data)),
    };
  }
)(StudentsList);

export default StudentsContainer;
```

Ce fichier pourrait ne pas exister. Dans ce cas, StudentsList aurait intégré ceci avant sa dernière ligne :

```js
StudentsList = connect(
  function mapStateToProps(state) {
    return { students: state };
  },
  function mapDispatchToProps(dispatch) {
    return {
        toggleForm: id      => dispatch(toggleForm(id)),
        updateForm: data    => dispatch(updateForm(data)),
        addStudent: data    => dispatch(addStudent(data)),
        deleteStudent: data => dispatch(deleteStudent(data)),
    };
  }
)(StudentsList);
```

et app.js aurait importé StudentsList plutôt que StudentContainer. Cependant, les fonctions purement Redux m'apparaissent plus clairement en extrayant le container dans un fichier à part.

Que fait ce conteneur ? Il possède deux fonctions Redux :

* mapStateToProps : il prend l'état \(ou une partie de l'état en fonction du besoin\) de l'application \(ici l'objet contenant les élèves\) et le transmet à StudentsList sous forme de propriété. C'est pour cela que dans StudentsList, on reçoit students dans les propriétés : `const { students, ... } = props;`
* mapDispatchToProps : cette fonction passe des fonctions sous forme de propriétés à StudentsList. C'est la suite des propriétés envoyées à StudentsList : `const { ..., toggleForm, updateForm, addStudent, deleteStudent } = props;`Elle prend en paramère la fonction dispatch de Redux, chargée d'informer le store qu'un changement d'état se produit et qu'il faut donc appeler l'action adéquate \(contenue dans le créateur d'actions - actions.js\). La fonction mapDispatchToProps se charge donc de faire correspondre chaque propriété passée à StudentsLists à la fonction issue de action.js \(qui lui même appelera la bonne fonction dans le reducer\). Par exemple, au clic à la création d'un élève, la propriété addStudent contient une référence à la fonction addStudent de action.js. Ce lien est fait ici, dans mapDispatchToProps. Ainsi, à la soumission du formulaire de création d'un élève, la propriété addStudent de StudentsList est appelée. C'est en fait la fonction addStudent de actions.js qui sera déclenchée \(et in fine, le cas "ADD\_STUDENT" du reducer, permettant le changement réel de l'état de l'appliction, permettant d'augmenter l'objet gérant les élèves d'un enregistrement\).

  Ouf ! Sacré cheminement, mais il le fallait pour faire le tour : les props envoyées à StudentsList permettent la connexion à Redux. A chaque changement d'état à appliquer, une fonction embarquée dans une propriété est déclenchée depuis un composant \(Student par exemple\). Le lien entre cette fonction à appeler et cette propriété est faite dans le container. Le container permet donc le dispatch vers l'action adéquate, action présente dans le fichier actions.js. Cette action contient un type \(qui doit être unique\). Redux appelle les reducers attachés au store. Le reducer contenant le même type que l'action appelée va exécuter sa fonction pure correspondante. Le changement d'état va déclencher un affichage dans la vue. C'est le fameux one-way data binding de Redux.

Pour faire plus propre, on peut ajouter un lien vers la gestion des élèves dans le composant Header \(layout/Header.js\):

```js
import React from 'react';
import { Nav, Navbar, NavItem } from 'react-bootstrap';
import { LinkContainer } from 'react-router-bootstrap';

const Header = () => {
    return (
        <Navbar>
            <Navbar.Header>
                <Navbar.Brand>
                    <a href="/">Site de tutoriel</a>
                </Navbar.Brand>
                <Navbar.Toggle />
            </Navbar.Header>
            <Navbar.Collapse>
                <Nav>
                    <LinkContainer to="home"><NavItem>Accueil</NavItem></LinkContainer>
                    <LinkContainer to="page1"><NavItem>Menu 1</NavItem></LinkContainer>
                    <LinkContainer to="page2"><NavItem>Menu 2</NavItem></LinkContainer>
                    <LinkContainer to="students"><NavItem>Elèves</NavItem></LinkContainer>
                </Nav>
            </Navbar.Collapse>
        </Navbar>
    );
};

export default Header;
```

J'ai aussi ajouté quelques éléments css, notamment une animation pour l'apparition des informations d'un élève \(.toggle ~ form\). Comme quoi, pas besoin de JS parfois ;\)

Voici le code du fichier app/Resources/scss/style.scss :

```css
$icon-font-path: "~bootstrap-sass/assets/fonts/bootstrap/";
@import "~bootstrap-sass/assets/stylesheets/bootstrap";

html, body {
  margin: 10px;
  font-size: 20px;
}

ul {
  padding: 0;
  list-style-type: none;
}

.btn {
    margin-left: 5px;
}

.row {
    margin-top: 5px;
}

.toggle {
    display: none;
}

.toggle ~ form {
    position: relative;
    left: 300px;
    transition: left 500ms cubic-bezier(0.17, 0.04, 0.03, 0.94);
}

.toggle:checked ~ form {
    left: 0;
}

.margin-bottom-5 {
    margin-bottom: 5px;
}

.alert{
    padding: 5px;
    margin: 10px 0;
}

.list-group-item {
    background-color: #fafafa;
}
.list-group-item:hover {
    background-color: #fff;
}
```

Lorsque l'on navigue sur l'url localhost:8000/students, on doit voir apparaître ceci :

![](/assets/Capture d’écran 2017-02-03 à 23.51.21.png)

On peut modifier, supprimer ou ajouter un élève désormais. Il y a même une validation basique sur le nombre de caractères à entrer pour les noms, ou la vérification qu'une note est bien numérique.

[^1]: C'était Rolland Garros, je me suis donc inspiré des 4 mousquetaires du tennis français et d'une célèbre tenniswoman de la même époque.

