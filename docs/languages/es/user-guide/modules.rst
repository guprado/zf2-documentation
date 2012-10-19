.. _user-guide.modules:

#######
Módulos
#######

Zend Framework 2 utiliza un sistema de módulos para organizar el código específico
para cada etapa en su correspondiente módulo. El módulo Application que provee la
aplicación esqueleto se utiliza para proveer la configuración de bootstrapping, error y
enrutamiento para toda la aplicación. Se utiliza habitualmente para proveer controladores
de nivel de aplicación para, se podría decir, la página de inicio de una aplicación, pero 
no vamos a utilizar la que viene por defecto en este tutorial, pues queremos que la página
de inicio sea la lista de albums, la cual va a vivir en nuestro módulo.

Vamos a poner todo nuestro código en el módulo Album, el cual va a contener nuestros
controladores, modelos, formularios y vistas. Además vamos a necesitar algunos archivos de
configuración.

Comencemos con las carpetas requeridas.

Configurando el módulo Album
----------------------------

Comience creando una carpeta llamada ``Album`` con los siguientes
subdirectorios para guardar los archivos del módulo:

.. code-block:: text

    zf2-tutorial/
        /module
            /Album
                /config
                /src
                    /Album
                        /Controller
                        /Form
                        /Model
                /view
                    /album
                        /album

Como puede ver, el módulo ``Album`` posee directorios separados para los diferentes
tipos de archivos que va a tener. Los archivos PHP que contienen clases con el
namespace ``Album`` viven en el directorio ``src/Album``, por lo que podemos tener
múltiples namespaces dentro de nuestro módulo cuando sea necesario. El directorio de vistas
tiene también un subdirectorio llamado ``album`` para las vistas de nuestro módulo.

----> Para poder cargar y configurar un módulo, Zend Framework 2 tiene un
``ModuleManager``.

In order to load and configure a module, Zend Framework 2 has a
``ModuleManager``. This will look for ``Module.php`` in the root of the module
directory (``module/Album``) and expect to find a class called ``Album\Module``
within it. That is, the classes within a given module will have the namespace of
the module’s name, which is the directory name of the module. 

Create ``Module.php`` in the ``Album`` module:

.. code-block:: php

    // module/Album/Module.php
    namespace Album;
    
    class Module
    {
        public function getAutoloaderConfig()
        {
            return array(
                'Zend\Loader\ClassMapAutoloader' => array(
                    __DIR__ . '/autoload_classmap.php',
                ),
                'Zend\Loader\StandardAutoloader' => array(
                    'namespaces' => array(
                        __NAMESPACE__ => __DIR__ . '/src/' . __NAMESPACE__,
                    ),
                ),
            );
        }
    
        public function getConfig()
        {
            return include __DIR__ . '/config/module.config.php';
        }
    }

The ``ModuleManager`` will call ``getAutoloaderConfig()`` and ``getConfig()``
automatically for us.

Autoloading files
^^^^^^^^^^^^^^^^

Our ``getAutoloaderConfig()`` method returns an array that is compatible with
ZF2’s ``AutoloaderFactory``. We configure it so that we add a class map file to
the ``ClassmapAutoloader`` and also add this module’s namespace to the
``StandardAutoloader``. The standard autoloader requires a namespace and the
path where to find the files for that namespace. It is PSR-0 compliant and so
classes map directly to files as per the `PSR-0 rules
<https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md>`_.

As we are in development, we don’t need to load files via the classmap, so we provide an empty array for the 
classmap autoloader. Create ``autoload_classmap.php`` with these contents:

.. code-block:: php

    <?php
    // module/Album/autoload_classmap.php:
    return array();

As this is an empty array, whenever the autoloader looks for a class within the
``Album`` namespace, it will fall back to the to ``StandardAutoloader`` for us.

.. note::

    Note that as we are using Composer, as an alternative, you could not implement
    ``getAutoloaderConfig()`` and instead add ``"Application":
    "module/Application/src"`` to the ``psr-0`` key in ``composer.json``. If you go
    this way, then you need to run ``php composer.phar update`` to update the
    composer autoloading files.

Configuration
-------------

Having registered the autoloader, let’s have a quick look at the ``getConfig()``
method in ``Album\Module``.  This method simply loads the
``config/module.config.php`` file.

Create the following configuration file for the ``Album`` module:

.. code-block:: php

    // module/Album/config/module.config.php:
    return array(
        'controllers' => array(
            'invokables' => array(
                'Album\Controller\Album' => 'Album\Controller\AlbumController',
            ),
        ),
        'view_manager' => array(
            'template_path_stack' => array(
                'album' => __DIR__ . '/../view',
            ),
        ),
    );

The config information is passed to the relevant components by the
``ServiceManager``.  We need two initial sections: ``controller`` and
``view_manager``. The controller section provides a list of all the controllers
provided by the module. We will need one controller, ``AlbumController``, which
we’ll reference as ``Album\Controller\Album``. The controller key must
be unique across all modules, so we prefix it with our module name.

Within the ``view_manager`` section, we add our view directory to the
``TemplatePathStack`` configuration. This will allow it to find the view scripts for
the ``Album`` module that are stored in our ``views/`` directory.

Informing the application about our new module
----------------------------------------------

We now need to tell the ``ModuleManager`` that this new module exists. This is done
in the application’s ``config/application.config.php`` file which is provided by the
skeleton application. Update this file so that its ``modules`` section contains the
``Album`` module as well, so the file now looks like this:

(Changes required are highlighted using comments.)

.. code-block:: php

    // config/application.config.php:
    return array(
        'modules' => array(
            'Application',
            'Album',                  // <-- Add this line
        ),
        'module_listener_options' => array( 
            'config_glob_paths'    => array(
                'config/autoload/{,*.}{global,local}.php',
            ),
            'module_paths' => array(
                './module',
                './vendor',
            ),
        ),
    );

As you can see, we have added our ``Album`` module into the list of modules
after the ``Application`` module.    

We have now set up the module ready for putting our custom code into it.
