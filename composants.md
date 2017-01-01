# Composants

Passons maintenant à la création des composants nécessaires à la gestion des élèves.

Le premier sera un composant d'affichage, un simple tableau affichant plusieurs fois un sous composant : les élèves. Puis un troisième composant permettra la création, la modification et la suppression d'un élève. Ce sera le plus complet et le plus complexe, introduisant l'usage de Redux-Form.

Voici le composant affichant le tableau :

```js
import React from 'react';

export default const StudentsList = props => {

  const { students, toggleForm, updateForm, addStudent, deleteStudent } = props;

  return (
    <div>
        <div className="col-lg-12">
        
          <ul className="list-group">
            {students.reducer.map(student => (
              <li key={student.get("id")} className="list-group-item clearfix">
                <Student item={student.toJS()} toggleForm={toggleForm} updateForm={updateForm} deleteStudent={deleteStudent}/>
              </li>
            ))}
          </ul>
          
          <hr/>
          
          <label>Ajouter un élève</label>
          <StudentForm key="new"
           formKey="new"
           saveForm={addStudent}
           /> 
           
        </div>
    </div>
  );
}
```

Les propriétés passées à notre composant sont "déstructurées" comme ceci : 

```js
const { students, toggleForm, updateForm, addStudent, deleteStudent } = props;
```

Pour ceux qui ne connaissent pas cette fonctionnalité ES6 : [http://putaindecode.io/fr/articles/js/es2015/destructuring/](http://putaindecode.io/fr/articles/js/es2015/destructuring/) \(je refais de la pub pour ce site, mais on y trouve tellement d'articles intéressants et clairs que je ne peux que le recommander\).

Puis dans une liste ul, on boucle sur la liste des élèves et on appelle le composant Student.

Enfin, en bas, on affiche le composant StudentForm, qui affichera le formulaire d'ajout d'élève.



