# Composants

Passons maintenant à la création des composants nécessaires à la gestion des élèves.

## Tableau

Le premier sera un composant d'affichage, un simple tableau affichant plusieurs fois un sous-composant : les élèves. Puis un troisième composant permettra la création, la modification et la suppression d'un élève. Ce sera le plus complet et le plus complexe, introduisant l'usage de Redux-Form.

Voici le composant affichant le tableau :

```js
import React from 'react';
import Student from './Student';
import StudentForm from './StudentForm';

let StudentsList = props => {

  const { students, toggleForm, updateForm, addStudent, deleteStudent } = props;

  return (
    <div className='col-lg-12'>

      <ul className='list-group'>
        {students.reducer.map(student => (
          <li key={student.get('id')} className='list-group-item clearfix'>
            <Student item={student.toJS()} toggleForm={toggleForm} updateForm={updateForm} deleteStudent={deleteStudent}/>
          </li>
        ))}
      </ul>

      <hr/>

      <label>Ajouter un élève</label>
      <StudentForm key='new'
       formKey='new'
       saveForm={addStudent}
       /> 

    </div>
  );
}

export default StudentsList;
```

Les propriétés passées à notre composant sont "déstructurées" comme ceci :

```js
const { students, toggleForm, updateForm, addStudent, deleteStudent } = props;
```

Pour ceux qui ne connaissent pas cette fonctionnalité ES6 : [http://putaindecode.io/fr/articles/js/es2015/destructuring/](http://putaindecode.io/fr/articles/js/es2015/destructuring/) \(je refais de la pub pour ce site, mais on y trouve tellement d'articles intéressants et clairs que je ne peux que le recommander\).

Puis dans une liste &lt;ul&gt;, on boucle sur la liste des élèves et on appelle le composant Student.

Enfin, en bas, on affiche le composant StudentForm, qui affichera le formulaire d'ajout d'élève.

## Elève

```js
import React from 'react';
import StudentForm from './StudentForm';

const Student = props => {

  const { item, toggleForm, updateForm, deleteStudent } = props;

  return (
      <span>
        <div class='container'>
            <div class='row'>

                    {item.firstName + ' ' + item.lastName}&nbsp;
                    <input type='checkbox' id={'toggle' + item.id} className='toggle' />
                    <label htmlFor={'toggle' + item.id} className='btn btn-primary pull-right margin-bottom-5' onClick={()=>toggleForm(item.id)}>
                        <span className='glyphicon glyphicon-pencil'></span>&nbsp;Editer
                    </label>

                    { item.isVisible ? <StudentForm key={item.id}
                           formKey={String(item.id)}
                           initialValues={item} 
                           saveForm={updateForm}
                           deleteStudent={deleteStudent}
                           /> : null 
                    }

            </div>
        </div>
      </span>
  );
}

export default Student;
```

Pour chaque élève, plusieurs propriétés sont passées au composant : item, toggleForm, updateForm, deleteStudent.

item contient les informations de l'élève : nom, prénom, id, ...

toggleForm, updateForm et deleteStudent sont des fonctions qui seront utilisées en fonction d'évènements.

## Formulaire

```js
import React, { PropTypes } from 'react';
import { reduxForm } from 'redux-form';

const validate = values => {
  const errors = {};

  if (!values.firstName) {
    errors.firstName = 'Le prénom est requis';
  } else if (values.firstName.length < 3) {
    errors.firstName = 'Le prénom doit être d\'au moins 3 caractères';
  }

  if (!values.lastName) {
    errors.lastName = 'Le nom est requis';
  } else if (values.lastName.length < 3) {
    errors.lastName = 'Le nom doit être d\'au moins 3 caractères';
  }

 if (isNaN(Number(values.average)) && values.average) {
    errors.average = 'La moyenne doit être numérique';
  } else if (Number(values.average) < 0) {
    errors.average = 'La moyenne doit être supérieure à 0';
  }

  return errors;
}

let StudentForm = props => {

    const save = (data) => {
        saveForm(data);
        if(!id.value) { 
            resetForm(); 
        }
    }

    const {fields: {id, firstName, lastName, level, average, isVisible, isUpdated}, handleSubmit, resetForm, saveForm, deleteStudent} = props;

    return (
      <form onSubmit={handleSubmit(save)}>
            <div className='row'>
              <div className='col-lg-2 col-md-2 col-sm-2 col-xs-4'><label>Prénom</label></div>
              <div className='col-lg-4 col-md-4 col-sm-4 col-xs-8'><input type='text' placeholder='Prénom élève' {...firstName}/></div>
              <div className='col-lg-2 col-md-2 col-sm-2 col-xs-4'><label>Nom</label></div>
              <div className='col-lg-4 col-md-4 col-sm-4 col-xs-8'><input type='text' placeholder='Nom élève' {...lastName}/></div>


            </div>
            <div className='row'>
              <div className='col-lg-2 col-md-2 col-sm-2 col-xs-4'><label>Niveau</label></div>
              <div className='col-lg-4 col-md-4 col-sm-4 col-xs-8'><input type='text' placeholder='6e, 5e, ...' {...level}/></div>
              <div className='col-lg-2 col-md-2 col-sm-2 col-xs-4'><label>Moyenne</label></div>
              <div className='col-lg-4 col-md-4 col-sm-4 col-xs-8'><input type='text' placeholder='Note sur 20' {...average}/></div>

            </div>
            <div className='row'>
                <div className='col-lg-12 col-md-12 col-sm-12 col-xs-12'>
                    <button className='btn btn-primary pull-right' type='submit'>Enregistrer</button>
                    {id.value ? <span className='btn btn-danger pull-right' onClick={()=>deleteStudent(id.value)}>Supprimer</span> : null }
                </div>
            </div>
            {firstName.touched && firstName.error && <div className='col-lg-12 col-md-12 col-sm-12 col-xs-12 text-danger'>{firstName.error}</div>}
            {lastName.touched && lastName.error && <div className='col-lg-12 col-md-12 col-sm-12 col-xs-12 text-danger'>{lastName.error}</div>}
            {average.touched && average.error && <div className='col-lg-12 col-md-12 col-sm-12 col-xs-12 text-danger'>{average.error}</div>}
            { isUpdated.value ? <div className='alert alert-success'>Modification enregistrée</div> : null }
      </form>
    );
};


StudentForm.propTypes = {
    fields: PropTypes.object.isRequired,
    handleSubmit: PropTypes.func.isRequired,
    saveForm: PropTypes.func.isRequired,
    deleteStudent: PropTypes.func
}

StudentForm = reduxForm({
  form: 'student',
  fields: ['id', 'firstName', 'lastName', 'level', 'average', 'isVisible', 'isUpdated'],
  validate 
})(StudentForm);

export default StudentForm;
```

Un peu plus long ce fichier, mais c'est parce qu'on utilise Redux-Form, qui est un moyen de gérer l'état d'un formulaire avec Redux. Il impose une certaine structure notamment une fonction de validation, qui contient les règles pour chaque champ à valider :

```js
const validate = values => {
  const errors = {};

  ...

  return errors;
}
```

Puis le composant en lui-même, qui contient les champs et des fonctions natives de Redux-Form : handleSubmit et resetForm \(voir [la documentation](http://redux-form.com/5.1.0/#/api/props) pour en savoir plus\).

Enfin, le rattachement du composant à Redux-Form se fait de cette manière :

```js
StudentForm = reduxForm({
  form: 'student',
  fields: ['id', 'firstName', 'lastName', 'level', 'average', 'isVisible', 'isUpdated'],
  validate 
})(StudentForm);
```

Ceci va permettre de connecter le formulaire à Redux pour gérer son état \(validation, soumission, réinitialisation\).

Une fois les composants définis, il manque la mécanique de Redux : le reducer, le container et les actions. Voyons tout ceci dans la dernière étape à examiner.

