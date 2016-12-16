# La gestion de l'état

C'est quoi cet état et pourquoi le gérer ?

Dans une application web moderne, nous gérons des tas d'éléments comme :

* la connexion utilisateur
* des formulaires
* des droits utilisateur sur des pages
* l'ajout, la modification et l'affichage de données \(le fameux [CRUD](https://fr.wikipedia.org/wiki/CRUD)\)
* ...

Ces éléments sont contenus dans des objets au sein d'une application React. Comment savoir dans toutes les pages de l'application si :

* un utilisateur est connecté
* un formulaire est validé
* l'utilisateur en cours a le droit d'accès à la page
* la modification d'une donnée en base est bien répercutée en front pour être réactualisée
* .... ?

Il faut gérer l'état de ces objets \(state management en anglais\) et le conserver tout au long de la vie de l'application. C'est là qu'entrent en jeu des architectures pour traiter les changements d'états. 


En plus de React - qui est assimilé au [V dans pattern design MVC](https://facebook.github.io/react/docs/why-react.html) - Facebook a créé l'architecture [Flux](https://facebook.github.io/flux/docs/overview.html). Elle utilise le concept de flux unidirectionnel de données \(contrairement à Angular ou Vue qui sont sur un double-way data binding\) : cela signifie que toutes les données d'une application suivent le même modèle de cycle de vie, rendant la logique dans l'app plus prévisible et simple à comprendre. Cela encourage aussi l'unicité des données, pour ne pas avoir des versions multiples et indépendantes de la même donnée.

Cette architecture est structurée ainsi:

* Actions – Des méthodes qui vont passer des données au Dispatcher
* Dispatcher – Reçoit les actions et renvoit des charges utiles \(payloads\) à des fonctions de rappel \(callbacks\) enregistrées au préalable
* Stores – Contient l'état de l'application et la logique des fonctions de rappel enregistrées dans le Dispatcher
* Controller Views – Des composants React qui chargent l'état depuis le ou les Stores et le passent à des composants enfants.

Un article bien clair à ce sujet : [http://putaindecode.io/fr/articles/js/flux/](http://putaindecode.io/fr/articles/js/flux/)

Tout ceci peut paraître flou encore pour l'instant, mais avec l'utilisation de Redux, les choses s'éclairciront.





