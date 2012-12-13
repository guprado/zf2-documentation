.. _user-guide-forms-and-actions:

######################
Formularios y acciones
######################

Añadiendo nuevos albums
-----------------------

Ahora podemos programar la funcionalidad para añadir nuevos albums. Hay dos cosas que realizar
en esta parte:

* Mostrar un formulario para que el usuario introduzca información
* Procesar la información del formulario y guardarla en la base de datos
  
Para hacer esto utilizamos ``Zend\Form``. El componente ``Zend\Form`` maneja el formulario,
y para validar la información añadimos un ``Zend\InputFilter`` a nuestra entidad ``Album``.
Comenzamos creando una nueva clase ``Album\Form\AlbumForm`` que extienda de
``Zend\Form\Form`` para definir nuestro formulario. La clase está almacenada en el
archivo ``AlbumForm.php`` dentro del directorio ``module/Album/src/Album/Form``.

Cree este archivo ahora:

.. code-block:: php

    // module/Album/src/Album/Form/AlbumForm.php:
    namespace Album\Form;

    use Zend\Form\Form;

    class AlbumForm extends Form
    {
        public function __construct($name = null)
        {
            // we want to ignore the name passed
            parent::__construct('album');
            $this->setAttribute('method', 'post');
            $this->add(array(
                'name' => 'id',
                'attributes' => array(
                    'type'  => 'hidden',
                ),
            ));
            $this->add(array(
                'name' => 'artist',
                'attributes' => array(
                    'type'  => 'text',
                ),
                'options' => array(
                    'label' => 'Artist',
                ),
            ));
            $this->add(array(
                'name' => 'title',
                'attributes' => array(
                    'type'  => 'text',
                ),
                'options' => array(
                    'label' => 'Title',
                ),
            ));
            $this->add(array(
                'name' => 'submit',
                'attributes' => array(
                    'type'  => 'submit',
                    'value' => 'Go',
                    'id' => 'submitbutton',
                ),        
            ));
        }
    }

Dentro del constructor de ``AlbumForm``, establecemos el nombre para llamar al constructor
de la clase padre, establecemos el método y después creamos cuatro elementos de formulario para 
id, artist, title y el botón submit. Para cada elemento establecemos varios atributos y opciones,
incluida la etiqueta a mostrar.

También necesitamos establecer validación para este formulario. En Zend Framework 2 esto se hace
utilizando un filtro de entrada que puede tanto ser autónomo como estar dentro de cualquier clase
que implemente ``InputFilterAwareInterface``, como una entidad del modelo. Vamos a
añadir el filtro de entrada a nuestra entidad ``Album``:

.. code-block:: php

    // module/Album/src/Album/Model/Album.php:
    namespace Album\Model;

    use Zend\InputFilter\Factory as InputFactory;
    use Zend\InputFilter\InputFilter;
    use Zend\InputFilter\InputFilterAwareInterface;
    use Zend\InputFilter\InputFilterInterface;

    class Album implements InputFilterAwareInterface
    {
        public $id;
        public $artist;
        public $title;
        protected $inputFilter;

        public function exchangeArray($data)
        {
            $this->id     = (isset($data['id']))     ? $data['id']     : null;
            $this->artist = (isset($data['artist'])) ? $data['artist'] : null;
            $this->title  = (isset($data['title']))  ? $data['title']  : null;
        }

        public function setInputFilter(InputFilterInterface $inputFilter)
        {
            throw new \Exception("Not used");
        }

        public function getInputFilter()
        {
            if (!$this->inputFilter) {
                $inputFilter = new InputFilter();
                $factory     = new InputFactory();

                $inputFilter->add($factory->createInput(array(
                    'name'     => 'id',
                    'required' => true,
                    'filters'  => array(
                        array('name' => 'Int'),
                    ),            
                )));

                $inputFilter->add($factory->createInput(array(
                    'name'     => 'artist',
                    'required' => true,
                    'filters'  => array(
                        array('name' => 'StripTags'),
                        array('name' => 'StringTrim'),
                    ),
                    'validators' => array(
                        array(
                            'name'    => 'StringLength',
                            'options' => array(
                                'encoding' => 'UTF-8',
                                'min'      => 1,
                                'max'      => 100,
                            ),
                        ),
                    ),
                )));

                $inputFilter->add($factory->createInput(array(
                    'name'     => 'title',
                    'required' => true,
                    'filters'  => array(
                        array('name' => 'StripTags'),
                        array('name' => 'StringTrim'),
                    ),
                    'validators' => array(
                        array(
                            'name'    => 'StringLength',
                            'options' => array(
                                'encoding' => 'UTF-8',
                                'min'      => 1,
                                'max'      => 100,
                            ),
                        ),
                    ),
                )));

                $this->inputFilter = $inputFilter;        
            }

            return $this->inputFilter;
        }
    }

``InputFilterAwareInterface`` define dos métodos: ``setInputFilter()`` y 
``getInputFilter()``. Sólo necesitamos implementar ``getInputFilter()`` y
simplemente lanzamos una excepción en ``setInputFilter()``.

Dentro de ``getInputFilter()`` instanciamos un ``InputFilter`` y añadimos los
input que necesitemos. Añadimos un input para cada propiedad que queramos
filtrar o validar. Para el campo ``id`` añadimos un filtro ``Int`` dado que sólo
necesitamos enteros. Para los elementos de texto añadimos dos filtros, ``StripTags`` y
``StringTrim`` para eliminar HTML no deseado y espacio en blanco innecesario. También establecemos
que sean *required* y añadimos un validador ``StringLength`` para asegurarnos de que
el usuario no introduce más caracteres de los que podemos almacenar en la base de datos.

Ahora necesitamos obtener el formulario para mostrar y después procesar la petición.
Esto se realiza dentro de ``addAction()`` de ``AlbumController``:

.. code-block:: php

    // module/Album/src/Album/Controller/AlbumController.php:

    //...
    use Zend\Mvc\Controller\AbstractActionController;
    use Zend\View\Model\ViewModel;
    use Album\Model\Album;          // <-- Add this import
    use Album\Form\AlbumForm;       // <-- Add this import
    //...

        // Add content to this method:
        public function addAction()
        {
            $form = new AlbumForm();
            $form->get('submit')->setValue('Add');

            $request = $this->getRequest();
            if ($request->isPost()) {
                $album = new Album();
                $form->setInputFilter($album->getInputFilter());
                $form->setData($request->getPost());

                if ($form->isValid()) {
                    $album->exchangeArray($form->getData());
                    $this->getAlbumTable()->saveAlbum($album);

                    // Redirect to list of albums
                    return $this->redirect()->toRoute('album');
                }
            }
            return array('form' => $form);
        }
    //...

Tras añadir el ``AlbumForm`` a la lista de uso, implementamos ``addAction()``.
Vamos a analizar el código de ``addAction()`` con algo más de detalle:

.. code-block:: php

    $form = new AlbumForm();
    $form->submit->setValue('Add');

Instanciamos `AlbumForm` y damos valor “Add” a la etiqueta del botón submit. 
Hacemos esto aquí dado que querremos reutilizar el formulario cuando editemos un album y utilizaremos
una etiqueta diferente.

.. code-block:: php

    $request = $this->getRequest();
    if ($request->isPost()) {
        $album = new Album();
        $form->setInputFilter($album->getInputFilter());
        $form->setData($request->getPost());
        if ($form->isValid()) {

Si el método ``isPost()`` del objeto ``Request`` se evalua a true, entonces el formulario ha sido
entregado y establecemos el filtro de entrada del formulario desde una instancia de album. Entonces
pasamos los datos enviados al formulario y comprobamos si es válido utilizando la función miembro
``isValid()`` del formulario.

.. code-block:: php

    $album->exchangeArray($form->getData());
    $this->getAlbumTable()->saveAlbum($album);

Si el formulario es válido, obtenemos los datos del formulario y los almacenamos en el
modelio utilizando ``saveAlbum()``.

.. code-block:: php

    // Redirect to list of albums
    return $this->redirect()->toRoute('album');

Después de haber guardado la nueva fila de album, hacemos una redirección a la lista de albums
utilizando el plugin ``Redirect`` del controlador.

.. code-block:: php

    return array('form' => $form);

Finalmente, devolvemos las variables que queremos asignar a la vista. En este
caso, solamente el objeto formulario. Note que Zend Framework 2 también le permite simplemente
devolver un array que contenga las variables que asignar a la vista y este
creará un ``ViewModel`` para usted él solo. Esto ahorra teclear un poco más.

Ahora necesitamos representar el formulario en el script de vista add.phtml:

.. code-block:: php

    <?php
    // module/Album/view/album/album/add.phtml:

    $title = 'Add new album';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>
    <?php
    $form = $this->form;
    $form->setAttribute('action', $this->url('album', array('action' => 'add')));
    $form->prepare();

    echo $this->form()->openTag($form);
    echo $this->formHidden($form->get('id'));
    echo $this->formRow($form->get('title'));
    echo $this->formRow($form->get('artist'));
    echo $this->formSubmit($form->get('submit'));
    echo $this->form()->closeTag();

De nuevo mostramos un título como antes y entonces representamos el formulario. Zend Framework
proporciona algunos métodos de ayuda en las vistas para hacer esto un poco más fácil. El método de ayuda
``form()`` tiene dos métodos ``openTag()`` y ``closeTag()`` que utilizamos para abrir y 
cerrar el formulario. Entonces, para cada elemento con etiqueta, podemos utilizar ``formRow()``,
pero para los dos elementos autónomos utilizamos ``formHidden()`` y 
``formSubmit()``.

.. image:: ../images/user-guide.forms-and-actions.add-album-form.png
    :width: 940 px

Ahora debería ser capaz de utilizar el enlace “Add new album” en la página de inicio de la
aplicación para añadir un nuevo disco.

Editing an album
----------------

Editar un album es casi idéntico a añadirlo, por lo que el código es muy similar.
Esta vez utilizamos ``editAction()`` en el ``AlbumController``:

.. code-block:: php

    // module/Album/src/Album/AlbumController.php:
    //...

        // Add content to this method:
        public function editAction()
        {
            $id = (int) $this->params()->fromRoute('id', 0);
            if (!$id) {
                return $this->redirect()->toRoute('album', array(
                    'action' => 'add'
                ));
            }
            $album = $this->getAlbumTable()->getAlbum($id);

            $form  = new AlbumForm();
            $form->bind($album);
            $form->get('submit')->setAttribute('value', 'Edit');
            
            $request = $this->getRequest();
            if ($request->isPost()) {
                $form->setInputFilter($album->getInputFilter());
                $form->setData($request->getPost());

                if ($form->isValid()) {
                    $this->getAlbumTable()->saveAlbum($album);

                    // Redirect to list of albums
                    return $this->redirect()->toRoute('album');
                }
            }

            return array(
                'id' => $id,
                'form' => $form,
            );
        }
    //...

Este código debería parecerle confortablemente similar. Echemos un vistazo a las diferencias
con añadir un album. En primer lugar, buscamos el ``id`` que hay en la ruta
y lo utilizamos para cargar el album que queremos utilizar:

.. code-block:: php

    $id = (int) $this->params()->fromRoute('id', 0);
    if (!$id) {
        return $this->redirect()->toRoute('album', array(
            'action' => 'add'
        ));
    }
    $album = $this->getAlbumTable()->getAlbum($id);

``params`` es un plugin del controlador que proporciona una vía conveniente para recuperar
parámetros de la ruta. Lo utilizamos para recuperar el ``id`` de la
ruta que creamos en el archivo ``module.config.php`` del módulo. Si el ``id`` es cero,
entonces redirigimos a la acción añadir. En otro caso, seguimos tomando la entidad
album de la base de datos.

.. code-block:: php

    $form = new AlbumForm();
    $form->bind($album);
    $form->get('submit')->setAttribute('value', 'Edit');

El método ``bind()`` del formulario "conecta" el modelo con el formulario. Esto se utiliza en dos
vías:

# Cuando se muestra el formulario, los valores iniciales de cada elemento son extraídos 
  del modelo.
# Después de una validación exitosa en isValid(), los datos del formulario son devueltos 
  al modelo.

Estas operaciones se hacen utilizando un objeto hydrator. Hay un número de
hydrators, pero por defecto es ``Zend\Stdlib\Hydrator\ArraySerializable``
que espera encontrar dos métodos en el modelo: ``getArrayCopy()`` y
``exchangeArray()``. Ya hemos escrito ``exchangeArray()`` en nuestra
entidad ``Album``, por lo que solo necesitamos escribir ``getArrayCopy()``:

.. code-block:: php

    // module/Album/src/Album/Model/Album.php:
    // ...
        public function exchangeArray($data)
        {
            $this->id     = (isset($data['id']))     ? $data['id']     : null;
            $this->artist = (isset($data['artist'])) ? $data['artist'] : null;
            $this->title  = (isset($data['title']))  ? $data['title']  : null;
        }

        // Add the following method:
        public function getArrayCopy()
        {
            return get_object_vars($this);
        }
    // ...

Como resultado de utilizar ``bind()`` con su hydrator, no necesitamos volver a poblar
los datos del formulario, ya se ha realizado, por lo que ya podemos
llamar al método ``saveAlbum()`` para guardar los cambios en la base de datos.

La plantilla de la vista, ``edit.phtml``, se ve muy similar a la vista para añadir un
album:

.. code-block:: php

    <?php
    // module/Album/view/album/album/edit.phtml:

    $title = 'Edit album';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>

    <?php
    $form = $this->form;
    $form->setAttribute('action', $this->url(
        'album', 
        array(
            'action' => 'edit',
            'id'     => $this->id,
        )
    ));
    $form->prepare();

    echo $this->form()->openTag($form);
    echo $this->formHidden($form->get('id'));
    echo $this->formRow($form->get('title'));
    echo $this->formRow($form->get('artist'));
    echo $this->formSubmit($form->get('submit'));
    echo $this->form()->closeTag();

/// SEGUIR AQUI

The only changes are to use the ‘Edit Album’ title and set the form’s action to
the ‘edit’ action too.

You should now be able to edit albums.

Deleting an album
-----------------

To round out our application, we need to add deletion. We have a Delete link
next to each album on our list page and the naïve approach would be to do a
delete when it’s clicked. This would be wrong. Remembering our HTTP spec, we
recall that you shouldn’t do an irreversible action using GET and should use
POST instead.

We shall show a confirmation form when the user clicks delete and if they then
click “yes”, we will do the deletion. As the form is trivial, we’ll code it
directly into our view (``Zend\Form`` is, after all, optional!).

Let’s start with the action code in ``AlbumController::deleteAction()``:

.. code-block:: php

    // module/Album/src/Album/AlbumController.php:
    //...
        // Add content to the following method:
        public function deleteAction()
        {
            $id = (int) $this->params()->fromRoute('id', 0);
            if (!$id) {
                return $this->redirect()->toRoute('album');
            }

            $request = $this->getRequest();
            if ($request->isPost()) {
                $del = $request->getPost('del', 'No');

                if ($del == 'Yes') {
                    $id = (int) $request->getPost('id');
                    $this->getAlbumTable()->deleteAlbum($id);
                }

                // Redirect to list of albums
                return $this->redirect()->toRoute('album');
            }

            return array(
                'id'    => $id,
                'album' => $this->getAlbumTable()->getAlbum($id)
            );
        }
    //...

As before, we get the ``id`` from the matched route,and check the request
object’s ``isPost()`` to determine whether to show the confirmation page or to
delete the album. We use the table object to delete the row using the
``deleteAlbum()`` method and then redirect back the list of albums. If the
request is not a POST, then we retrieve the correct database record and assign
to the view, along with the ``id``.

The view script is a simple form:

.. code-block:: php

    <?php
    // module/Album/view/album/album/delete.phtml:

    $title = 'Delete album';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>

    <p>Are you sure that you want to delete 
        '<?php echo $this->escapeHtml($album->title); ?>' by 
        '<?php echo $this->escapeHtml($album->artist); ?>'?
    </p>
    <?php 
    $url = $this->url('album', array(
        'action' => 'delete', 
        'id'     => $this->id,
    )); 
    ?>
    <form action="<?php echo $url; ?>" method="post">
    <div>
        <input type="hidden" name="id" value="<?php echo (int) $album->id; ?>" />
        <input type="submit" name="del" value="Yes" />
        <input type="submit" name="del" value="No" />
    </div>
    </form>

In this script, we display a confirmation message to the user and then a form
with "Yes" and "No" buttons. In the action, we checked specifically for the “Yes”
value when doing the deletion.

Ensuring that the home page displays the list of albums
-------------------------------------------------------

One final point. At the moment, the home page, http://zf2-tutorial.localhost/
doesn’t display the list of albums. 

This is due to a route set up in the ``Application`` module’s
``module.config.php``. To change it, open
``module/Application/config/module.config.php`` and find the home route:

.. code-block:: php

    'home' => array(
        'type' => 'Zend\Mvc\Router\Http\Literal',
        'options' => array(
            'route'    => '/',
            'defaults' => array(
                'controller' => 'Application\Controller\Index',
                'action'     => 'index',
            ),
        ),
    ),

Change the ``controller`` from ``Application\Controller\Index`` to
``Album\Controller\Album``:

.. code-block:: php

    'home' => array(
        'type' => 'Zend\Mvc\Router\Http\Literal',
        'options' => array(
            'route'    => '/',
            'defaults' => array(
                'controller' => 'Album\Controller\Album', // <-- change here
                'action'     => 'index',
            ),
        ),
    ),

That’s it - you now have a fully working application!
