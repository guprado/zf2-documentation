.. _user-guide.routing-and-controllers:

############################
Enrutamiento y controladores
############################

Vamos a construir un sistema de inventario muy simple para mostrar nuestra
colección de albums. La página de inicio va a listar nuestra colección y nos va a permitir añadir, editar y
eliminar albums. De ahí que las páginas requeridas sean las siguientes:

+-----------------+--------------------------------------------------------------+
| Página          | Descripción                                                  |
+=================+==============================================================+
| Inicio          | Mostrará el listado de discos y proveerá enlaces para        |
|                 | editarlos y modificarlos. Además, proveerá un enlace para    |
|                 | añadir nuevos albums.                                        |
+-----------------+--------------------------------------------------------------+
| Añadir album    | Esta página proveerá un formulario para añadir un album.     |
+-----------------+--------------------------------------------------------------+
| Editar album    | Esta página proveerá un formulario para editar un album.     |
+-----------------+--------------------------------------------------------------+
| Eliminar album  | Esta página confirmará que queremos eliminar un album y      |
|                 | después se eliminará.                                        |
+-----------------+--------------------------------------------------------------+

Antes de comenzar a montar nuestros archivos, es importante entender como espera el
framework que las páginas estén organizadas. Cada página de la aplicación es conocida como una
*acción* y las acciones están agrupadas dentro de *controladores* contenidos en *módulos*.
De esta forma, agrupará generalmente acciones relacionadas dentro de un mismo controlador;
por ejemplo, un controlador de noticias debería tener acciones ``current``, ``archived`` y ``view``.

Como tenemos cuatro páginas que están todas relacionadas con albums, vamos a agruparlas en un controlador
único ``AlbumController`` dentro de nuestro módulo ``Album`` como cuatro acciones.
Las cuatro acciones serán:

+-----------------+---------------------+------------+
| Página          | Controlador         | Acción     |
+=================+=====================+============+
| Inicio          | ``AlbumController`` | ``index``  |
+-----------------+---------------------+------------+
| Añadir album    | ``AlbumController`` | ``add``    |
+-----------------+---------------------+------------+
| Editar album    | ``AlbumController`` | ``edit``   |
+-----------------+---------------------+------------+
| Eliminar album  | ``AlbumController`` | ``delete`` |
+-----------------+---------------------+------------+

El mapeo de una URL a una acción particular se realiza utilizando rutas definidas
en el archivo ``module.config.php`` del módulo. Añadiremos una ruta para nuestras
acciones de album. Este es el archivo de configuración actualizado, con el nuevo código comentado.

.. code-block:: php

    // module/Album/config/module.config.php:
    return array(
        'controllers' => array(
            'invokables' => array(
                'Album\Controller\Album' => 'Album\Controller\AlbumController',
            ),
        ),
        
        // La siguiente sección es nueva y debería ser añadida a su fichero
        'router' => array(
            'routes' => array(
                'album' => array(
                    'type'    => 'segment',
                    'options' => array(
                        'route'    => '/album[/:action][/:id]',
                        'constraints' => array(
                            'action' => '[a-zA-Z][a-zA-Z0-9_-]*',
                            'id'     => '[0-9]+',
                        ),
                        'defaults' => array(
                            'controller' => 'Album\Controller\Album',
                            'action'     => 'index',
                        ),
                    ),
                ),
            ),
        ),

        'view_manager' => array(
            'template_path_stack' => array(
                'album' => __DIR__ . '/../view',
            ),
        ),
    );

// CONTINUAR AQUÍ

The name of the route is ‘album’ and has a type of ‘segment’. The segment route
allows us to specify placeholders in the URL pattern (route) that will be mapped
to named parameters in the matched route. In this case, the route is
**``/album[/:action][/:id]``** which will match any URL that starts with
``/album``. The next segment will be an optional action name, and then finally
the next segment will be mapped to an optional id. The square brackets indicate
that a segment is optional. The constraints section allows us to ensure that the
characters within a segment are as expected, so we have limited actions to
starting with a letter and then subsequent characters only being alphanumeric,
underscore or hyphen. We also limit the id to a number.

This route allows us to have the following URLs:

+---------------------+------------------------------+------------+
| URL                 | Page                         | Action     |
+=====================+==============================+============+
| ``/album``          | Home (list of albums)        | ``index``  |
+---------------------+------------------------------+------------+
| ``/album/add``      | Add new album                | ``add``    |
+---------------------+------------------------------+------------+
| ``/album/edit/2``   | Edit album with an id of 2   | ``edit``   |
+---------------------+------------------------------+------------+
| ``/album/delete/4`` | Delete album with an id of 4 | ``delete`` |
+---------------------+------------------------------+------------+

Create the controller
=====================

We are now ready to set up our controller. In Zend Framework 2, the controller
is a class that is generally called ``{Controller name}Controller``. Note that
``{Controller name}`` must start with a capital letter.  This class lives in a file
called ``{Controller name}Controller.php`` within the ``Controller`` directory for the
module. In our case that is ``module/Album/src/Album/Controller``. Each action is
a public method within the controller class that is named ``{action name}Action``.
In this case ``{action name}`` should start with a lower case letter.

.. note::

    This is by convention. Zend Framework 2 doesn’t provide many
    restrictions on controllers other than that they must implement the
    ``Zend\Stdlib\Dispatchable`` interface. The framework provides two abstract
    classes that do this for us: ``Zend\Mvc\Controller\AbstractActionController``
    and ``Zend\Mvc\Controller\AbstractRestfulController``. We’ll be using the
    standard ``AbstractActionController``, but if you’re intending to write a
    RESTful web service, ``AbstractRestfulController`` may be useful.

Let’s go ahead and create our controller class:

.. code-block:: php

    // module/Album/src/Album/Controller/AlbumController.php:
    namespace Album\Controller;

    use Zend\Mvc\Controller\AbstractActionController;
    use Zend\View\Model\ViewModel;
    
    class AlbumController extends AbstractActionController
    {
        public function indexAction()
        {
        }
    
        public function addAction()
        {
        }
    
        public function editAction()
        {
        }
    
        public function deleteAction()
        {
        }
    }

.. note::

    We have already informed the module about our controller in the
    ‘controller’ section of ``config/module.config.php``.

We have now set up the four actions that we want to use. They won’t work yet
until we set up the views. The URLs for each action are:

+--------------------------------------------+----------------------------------------------------+
| URL                                        | Method called                                      |
+============================================+====================================================+
| http://zf2-tutorial.localhost/album        | ``Album\Controller\AlbumController::indexAction``  |
+--------------------------------------------+----------------------------------------------------+
| http://zf2-tutorial.localhost/album/add    | ``Album\Controller\AlbumController::addAction``    |
+--------------------------------------------+----------------------------------------------------+
| http://zf2-tutorial.localhost/album/edit   | ``Album\Controller\AlbumController::editAction``   |
+--------------------------------------------+----------------------------------------------------+
| http://zf2-tutorial.localhost/album/delete | ``Album\Controller\AlbumController::deleteAction`` |
+--------------------------------------------+----------------------------------------------------+

We now have a working router and the actions are set up for each page of our
application.

It’s time to build the view and the model layer.

Initialise the view scripts
---------------------------

To integrate the view into our application all we need to do is create some view
script files. These files will be executed by the ``DefaultViewStrategy`` and will be
passed any variables or view models that are returned from the controller action
method. These view scripts are stored in our module’s views directory within a
directory named after the controller. Create these four empty files now:

* ``module/Album/view/album/album/index.phtml``
* ``module/Album/view/album/album/add.phtml``
* ``module/Album/view/album/album/edit.phtml``
* ``module/Album/view/album/album/delete.phtml``

We can now start filling everything in, starting with our database and models.
