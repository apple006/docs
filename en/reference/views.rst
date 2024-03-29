Using Views
===========

Views represent the user interface of your application. Views are often HTML files with embedded PHP code that perform tasks
related solely to the presentation of the data. Views handle the job of providing data to the web browser or other tool that
is used to make requests from your application.

:doc:`ManaPHP\\Mvc\\View` and :doc:`ManaPHP\\Mvc\\View\\Simple` are responsible for the managing the view layer of your MVC application.

Integrating Views with Controllers
----------------------------------
ManaPHP automatically passes the execution to the view component as soon as a particular controller has completed its cycle. The view component
will look in the views folder for a folder named as the same name of the last controller executed and then for a file named as the last action
executed. For instance, if a request is made to the URL *http://127.0.0.1/blog/posts/show/301*, ManaPHP will parse the URL as follows:

+-------------------+-----------+
| Server Address    | 127.0.0.1 |
+-------------------+-----------+
| ManaPHP Directory | blog      |
+-------------------+-----------+
| Controller        | posts     |
+-------------------+-----------+
| Action            | show      |
+-------------------+-----------+
| Parameter         | 301       |
+-------------------+-----------+

The dispatcher will look for a "PostsController" and its action "showAction". A simple controller file for this example:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Controller;

    class PostsController extends Controller
    {
        public function indexAction()
        {

        }

        public function showAction($postId)
        {
            // Pass the $postId parameter to the view
            $this->view->postId = $postId;
        }
    }

The setVar allows us to create view variables on demand so that they can be used in the view template. The example above demonstrates
how to pass the :code:`$postId` parameter to the respective view template.

Hierarchical Rendering
----------------------
:doc:`ManaPHP\\Mvc\\View` supports a hierarchy of files and is the default component for view rendering in ManaPHP.
This hierarchy allows for common layout points (commonly used views), as well as controller named folders defining respective view templates.

This component uses by default PHP itself as the template engine, therefore views should have the .phtml extension.
If the views directory is  *app/views* then view component will find automatically for these 3 view files.

+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Name              | File                          | Description                                                                                                                                                                                                              |
+===================+===============================+==========================================================================================================================================================================================================================+
| Action View       | app/views/posts/show.phtml    | This is the view related to the action. It only will be shown when the "show" action was executed.                                                                                                                       |
+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Controller Layout | app/views/layouts/posts.phtml | This is the view related to the controller. It only will be shown for every action executed within the controller "posts". All the code implemented in the layout will be reused for all the actions in this controller. |
+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Main Layout       | app/views/index.phtml         | This is main action it will be shown for every controller or action executed within the application.                                                                                                                     |
+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

You are not required to implement all of the files mentioned above. :doc:`ManaPHP\\Mvc\\View` will simply move to the
next view level in the hierarchy of files. If all three view files are implemented, they will be processed as follows:

.. code-block:: html+php

    <!-- app/views/posts/show.phtml -->

    <h3>This is show view!</h3>

    <p>I have received the parameter <?php echo $postId; ?></p>

.. code-block:: html+php

    <!-- app/views/layouts/posts.phtml -->

    <h2>This is the "posts" controller layout!</h2>

    <?php echo $this->getContent(); ?>

.. code-block:: html+php

    <!-- app/views/index.phtml -->
    <html>
        <head>
            <title>Example</title>
        </head>
        <body>

            <h1>This is main layout!</h1>

            <?php echo $this->getContent(); ?>

        </body>
    </html>

Note the lines where the method :code:`$this->getContent()` was called. This method instructs :doc:`ManaPHP\\Mvc\\View`
on where to inject the contents of the previous view executed in the hierarchy. For the example above, the output will be:

.. figure:: ../_static/img/views-1.png
   :align: center

The generated HTML by the request will be:

.. code-block:: html+php

    <!-- app/views/index.phtml -->
    <html>
        <head>
            <title>Example</title>
        </head>
        <body>

            <h1>This is main layout!</h1>

            <!-- app/views/layouts/posts.phtml -->

            <h2>This is the "posts" controller layout!</h2>

            <!-- app/views/posts/show.phtml -->

            <h3>This is show view!</h3>

            <p>I have received the parameter 101</p>

        </body>
    </html>

Picking Views
^^^^^^^^^^^^^
As mentioned above, when :doc:`ManaPHP\\Mvc\\View` is managed by :doc:`ManaPHP\\Mvc\\Application`
the view rendered is the one related with the last controller and action executed. You could override this by using the :code:`ManaPHP\Mvc\View::pick()` method:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Controller;

    class ProductsController extends Controller
    {
        public function listAction()
        {
            // Pick "views-dir/products/search" as view to render
            $this->view->pick("products/search");

            // Pick "views-dir/books/list" as view to render
            $this->view->pick(array('books'));

            // Pick "views-dir/products/search" as view to render
            $this->view->pick(array(1 => 'search'));
        }
    }

Disabling the view
^^^^^^^^^^^^^^^^^^
If your controller doesn't produce any output in the view (or not even have one) you may disable the view component avoiding unnecessary processing:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Controller;

    class UsersController extends Controller
    {
        public function closeSessionAction()
        {
            // Close session
            // ...

            // A HTTP Redirect
            $this->response->redirect('index/index');

            // Disable the view to avoid rendering
            $this->view->disable();
        }
    }

You can return a 'response' object to avoid disable the view manually:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Controller;

    class UsersController extends Controller
    {
        public function closeSessionAction()
        {
            // Close session
            // ...

            // A HTTP Redirect
            return $this->response->redirect('index/index');
        }
    }

Simple Rendering
----------------
:doc:`ManaPHP\\Mvc\\View\\Simple` is an alternative component to :doc:`ManaPHP\\Mvc\\View`.
It keeps most of the philosophy of :doc:`ManaPHP\\Mvc\\View` but lacks of a hierarchy of files which is, in fact,
the main feature of its counterpart.

This component allows the developer to have control of when a view is rendered and its location.
In addition, this component can leverage of view inheritance available in template engines such
as :doc:`Volt <volt>` and others.

The default component must be replaced in the service container:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\View\Simple as SimpleView;

    $di->set('view', function () {

        $view = new SimpleView();

        $view->setViewsDir('../app/views/');

        return $view;
    }, true);

Automatic rendering must be disabled in :doc:`ManaPHP\\Mvc\\Application <applications>` (if needed):

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Application;

    try {

        $application = new Application($di);

        $application->useImplicitView(false);

        echo $application->handle()->getContent();

    } catch (\Exception $e) {
        echo $e->getMessage();
    }

To render a view it's necessary to call the render method explicitly indicating the relative path to the view you want to display:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Controller;

    class PostsController extends \Controller
    {
        public function indexAction()
        {
            // Render 'views-dir/index.phtml'
            echo $this->view->render('index');

            // Render 'views-dir/posts/show.phtml'
            echo $this->view->render('posts/show');

            // Render 'views-dir/index.phtml' passing variables
            echo $this->view->render('index', array('posts' => Posts::find()));

            // Render 'views-dir/posts/show.phtml' passing variables
            echo $this->view->render('posts/show', array('posts' => Posts::find()));
        }
    }

This is different to :doc:`ManaPHP\\Mvc\\View` who's :code:`render()` method uses controllers and actions as parameters:

.. code-block:: php

    <?php

    $params = array('posts' => Posts::find());

    // ManaPHP\Mvc\View
    $view = new \ManaPHP\Mvc\View();
    echo $view->render('posts', 'show', $params);

    // ManaPHP\Mvc\View\Simple
    $simpleView = new \ManaPHP\Mvc\View\Simple();
    echo $simpleView->render('posts/show', $params);

Using Partials
--------------
Partial templates are another way of breaking the rendering process into simpler more manageable chunks that can be reused by different
parts of the application. With a partial, you can move the code for rendering a particular piece of a response to its own file.

One way to use partials is to treat them as the equivalent of subroutines: as a way to move details out of a view so that your code can be more easily understood.
For example, you might have a view that looks like this:

.. code-block:: html+php

    <div class="top"><?php $this->partial("shared/ad_banner"); ?></div>

    <div class="content">
        <h1>Robots</h1>

        <p>Check out our specials for robots:</p>
        ...
    </div>

    <div class="footer"><?php $this->partial("shared/footer"); ?></div>

Method partial() does accept a second parameter as an array of variables/parameters that only will exists in the scope of the partial:

.. code-block:: html+php

    <?php $this->partial("shared/ad_banner", array('id' => $site->id, 'size' => 'big')); ?>

Transfer values from the controller to views
--------------------------------------------
:doc:`ManaPHP\\Mvc\\View` is available in each controller using the view variable (:code:`$this->view`). You can
use that object to set variables directly to the view from a controller action by using the :code:`setVar()` method.

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Controller;

    class PostsController extends Controller
    {
        public function indexAction()
        {

        }

        public function showAction()
        {
            // Pass all the posts to the views
            $this->view->setVar(
                "posts",
                Posts::find()
            );

            // Using the magic setter
            $this->view->posts = Posts::find();

            // Passing more than one variable at the same time
            $this->view->setVars(
                array(
                    'title'   => $post->title,
                    'content' => $post->content
                )
            );
        }
    }

A variable with the name of the first parameter of setVar() will be created in the view, ready to be used. The variable can be of any type,
from a simple string, integer etc. variable to a more complex structure such as array, collection etc.

.. code-block:: html+php

    <div class="post">
    <?php

        foreach ($posts as $post) {
            echo "<h1>", $post->title, "</h1>";
        }

    ?>
    </div>

Using models in the view layer
------------------------------
Application models are always available at the view layer. The :doc:`ManaPHP\\Loader` will instantiate them at
runtime automatically:

.. code-block:: html+php

    <div class="categories">
    <?php

        foreach (Categories::find("status = 1") as $category) {
            echo "<span class='category'>", $category->name, "</span>";
        }

    ?>
    </div>

Although you may perform model manipulation operations such as insert() or update() in the view layer, it is not recommended since it is not possible to forward the execution flow to another controller in the case of an error or an exception.

Caching View Fragments
----------------------
Sometimes when you develop dynamic websites and some areas of them are not updated very often, the output is exactly
the same between requests. :doc:`ManaPHP\\Mvc\\View` offers caching a part or the whole
rendered output to increase performance.

:doc:`ManaPHP\\Mvc\\View` integrates with :doc:`ManaPHP\\Cache <cache>` to provide an easier way
to cache output fragments. You could manually set the cache handler or set a global handler:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Controller;

    class PostsController extends Controller
    {
        public function showAction()
        {
            // Cache the view using the default settings
            $this->view->cache(true);
        }

        public function showArticleAction()
        {
            // Cache this view for 1 hour
            $this->view->cache(
                array(
                    "lifetime" => 3600
                )
            );
        }

        public function resumeAction()
        {
            // Cache this view for 1 day with the key "resume-cache"
            $this->view->cache(
                array(
                    "lifetime" => 86400,
                    "key"      => "resume-cache"
                )
            );
        }

        public function downloadAction()
        {
            // Passing a custom service
            $this->view->cache(
                array(
                    "service"  => "myCache",
                    "lifetime" => 86400,
                    "key"      => "resume-cache"
                )
            );
        }
    }

When we do not define a key to the cache, the component automatically creates one using an MD5_ hash of the name of the controller and view currently being rendered in the format of "controller/view".
It is a good practice to define a key for each action so you can easily identify the cache associated with each view.

When the View component needs to cache something it will request a cache service from the services container.
The service name convention for this service is "viewCache":

.. code-block:: php

    <?php

    use ManaPHP\Cache\Frontend\Output as OutputFrontend;
    use ManaPHP\Cache\Backend\Memcache as MemcacheBackend;

    // Set the views cache service
    $di->set('viewCache', function () {

        // Cache data for one day by default
        $frontCache = new OutputFrontend(
            array(
                "lifetime" => 86400
            )
        );

        // Memcached connection settings
        $cache = new MemcacheBackend(
            $frontCache,
            array(
                "host" => "localhost",
                "port" => "11211"
            )
        );

        return $cache;
    });

.. highlights::
    The frontend must always be :doc:`ManaPHP\\Cache\\Frontend\\Output` and the service 'viewCache' must be registered as
    always open (not shared) in the services container (DI).

When using views, caching can be used to prevent controllers from needing to generate view data on each request.

To achieve this we must identify uniquely each cache with a key. First we verify that the cache does not exist or has
expired to make the calculations/queries to display data in the view:

.. code-block:: html+php

    <?php

    use ManaPHP\Mvc\Controller;

    class DownloadController extends Controller
    {
        public function indexAction()
        {
            // Check whether the cache with key "downloads" exists or has expired
            if ($this->view->getCache()->exists('downloads')) {

                // Query the latest downloads
                $latest = Downloads::find(
                    array(
                        'order' => 'created_at DESC'
                    )
                );

                $this->view->latest = $latest;
            }

            // Enable the cache with the same key "downloads"
            $this->view->cache(
                array(
                    'key' => 'downloads'
                )
            );
        }
    }

The `PHP alternative site`_ is an example of implementing the caching of fragments.

Template Engines
----------------
Template Engines help designers to create views without the use of a complicated syntax. ManaPHP includes a powerful and fast templating engine
called :doc:`Volt <volt>`.

Additionally, :doc:`ManaPHP\\Mvc\\View` allows you to use other template engines instead of plain PHP or Volt.

Using a different template engine, usually requires complex text parsing using external PHP libraries in order to generate the final output
for the user. This usually increases the number of resources that your application will use.

If an external template engine is used, :doc:`ManaPHP\\Mvc\\View` provides exactly the same view hierarchy and it's
still possible to access the API inside these templates with a little more effort.

This component uses adapters, these help ManaPHP to speak with those external template engines in a unified way, let's see how to do that integration.

Creating your own Template Engine Adapter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
There are many template engines, which you might want to integrate or create one of your own. The first step to start using an external template engine is create an adapter for it.

A template engine adapter is a class that acts as bridge between :doc:`ManaPHP\\Mvc\\View` and the template engine itself.
Usually it only needs two methods implemented: :code:`__construct()` and :code:`render()`. The first one receives the :doc:`ManaPHP\\Mvc\\View`
instance that creates the engine adapter and the DI container used by the application.

The method :code:`render()` accepts an absolute path to the view file and the view parameters set using :code:`$this->view->setVar()`. You could read or require it
when it's necessary.

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Engine;

    class MyTemplateAdapter extends Engine
    {
        /**
         * Adapter constructor
         *
         * @param \ManaPHP\Mvc\View $view
         * @param \ManaPHP\Di $di
         */
        public function __construct($view, $di)
        {
            // Initialize here the adapter
            parent::__construct($view, $di);
        }

        /**
         * Renders a view using the template engine
         *
         * @param string $path
         * @param array $params
         */
        public function render($path, $params)
        {
            // Access view
            $view    = $this->_view;

            // Access options
            $options = $this->_options;

            // Render the view
            // ...
        }
    }

Changing the Template Engine
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
You can replace or add more a template engine from the controller as follows:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Controller;

    class PostsController extends Controller
    {
        public function indexAction()
        {
            // Set the engine
            $this->view->registerEngines(
                array(
                    ".my-html" => "MyTemplateAdapter"
                )
            );
        }

        public function showAction()
        {
            // Using more than one template engine
            $this->view->registerEngines(
                array(
                    ".my-html" => 'MyTemplateAdapter',
                    ".phtml"   => 'ManaPHP\Mvc\View\Engine\Php'
                )
            );
        }
    }

You can replace the template engine completely or use more than one template engine at the same time. The method :code:`ManaPHP\Mvc\View::registerEngines()`
accepts an array containing data that define the template engines. The key of each engine is an extension that aids in distinguishing one from another.
Template files related to the particular engine must have those extensions.

The order that the template engines are defined with :code:`ManaPHP\Mvc\View::registerEngines()` defines the relevance of execution. If
:doc:`ManaPHP\\Mvc\\View` finds two views with the same name but different extensions, it will only render the first one.

If you want to register a template engine or a set of them for each request in the application. You could register it when the view service is created:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\View;

    // Setting up the view component
    $di->set('view', function () {

        $view = new View();

        // A trailing directory separator is required
        $view->setViewsDir('../app/views/');

        $view->registerEngines(
            array(
                ".my-html" => 'MyTemplateAdapter'
            )
        );

        return $view;
    }, true);

There are adapters available for several template engines on the `ManaPHP Incubator <https://github.com/ManaPHP/incubator/tree/master/Library/ManaPHP/Mvc/View/Engine>`_

Injecting services in View
--------------------------
Every view executed is included inside a :doc:`ManaPHP\\Di\\Injectable` instance, providing easy access
to the application's service container.

The following example shows how to write a jQuery `ajax request`_ using a URL with the framework conventions.
The service "url" (usually :doc:`ManaPHP\\Mvc\\Url <url>`) is injected in the view by accessing a property with the same name:

.. code-block:: html+php

    <script type="text/javascript">

    $.ajax({
        url: "<?php echo $this->url->get("cities/get"); ?>"
    })
    .done(function () {
        alert("Done!");
    });

    </script>

Stand-Alone Component
---------------------
All the components in ManaPHP can be used as *glue* components individually because they are loosely coupled to each other:

Hierarchical Rendering
^^^^^^^^^^^^^^^^^^^^^^
Using :doc:`ManaPHP\\Mvc\\View` in a stand-alone mode can be demonstrated below:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\View;

    $view = new View();

    // A trailing directory separator is required
    $view->setViewsDir("../app/views/");

    // Passing variables to the views, these will be created as local variables
    $view->setVar("someProducts", $products);
    $view->setVar("someFeatureEnabled", true);

    // Start the output buffering
    $view->start();

    // Render all the view hierarchy related to the view products/list.phtml
    $view->render("products", "list");

    // Finish the output buffering
    $view->finish();

    echo $view->getContent();

A short syntax is also available:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\View;

    $view = new View();

    echo $view->getRender('products', 'list',
        array(
            "someProducts"       => $products,
            "someFeatureEnabled" => true
        ),
        function ($view) {
            // Set any extra options here
            $view->setViewsDir("../app/views/");
            $view->setRenderLevel(View::LEVEL_LAYOUT);
        }
    );

Simple Rendering
^^^^^^^^^^^^^^^^
Using :doc:`ManaPHP\\Mvc\\View\\Simple` in a stand-alone mode can be demonstrated below:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\View\Simple as SimpleView;

    $view = new SimpleView();

    // A trailing directory separator is required
    $view->setViewsDir("../app/views/");

    // Render a view and return its contents as a string
    echo $view->render("templates/welcomeMail");

    // Render a view passing parameters
    echo $view->render(
        "templates/welcomeMail",
        array(
            'email'   => $email,
            'content' => $content
        )
    );

View Events
-----------
:doc:`ManaPHP\\Mvc\\View` and :doc:`ManaPHP\\Mvc\\View\\Simple` are able to send events to an :doc:`EventsManager <events>` if it is present. Events are triggered using the type "view". Some events when returning boolean false could stop the active operation. The following events are supported:

+----------------------+------------------------------------------------------------+---------------------+
| Event Name           | Triggered                                                  | Can stop operation? |
+======================+============================================================+=====================+
| beforeRender         | Triggered before starting the render process               | Yes                 |
+----------------------+------------------------------------------------------------+---------------------+
| beforeRenderView     | Triggered before rendering an existing view                | Yes                 |
+----------------------+------------------------------------------------------------+---------------------+
| afterRenderView      | Triggered after rendering an existing view                 | No                  |
+----------------------+------------------------------------------------------------+---------------------+
| afterRender          | Triggered after completing the render process              | No                  |
+----------------------+------------------------------------------------------------+---------------------+
| notFoundView         | Triggered when a view was not found                        | No                  |
+----------------------+------------------------------------------------------------+---------------------+

The following example demonstrates how to attach listeners to this component:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\View;
    use ManaPHP\Events\Manager as EventsManager;

    $di->set('view', function () {

        // Create an events manager
        $eventsManager = new EventsManager();

        // Attach a listener for type "view"
        $eventsManager->attach("view", function ($event, $view) {
            echo $event->getType(), ' - ', $view->getActiveRenderPath(), PHP_EOL;
        });

        $view = new View();
        $view->setViewsDir("../app/views/");

        // Bind the eventsManager to the view component
        $view->setEventsManager($eventsManager);

        return $view;

    }, true);

The following example shows how to create a plugin that clean/repair the HTML produced by the render process using Tidy_:

.. code-block:: php

    <?php

    class TidyPlugin
    {
        public function afterRender($event, $view)
        {
            $tidyConfig = array(
                'clean'          => true,
                'output-xhtml'   => true,
                'show-body-only' => true,
                'wrap'           => 0
            );

            $tidy = tidy_parse_string($view->getContent(), $tidyConfig, 'UTF8');
            $tidy->cleanRepair();

            $view->setContent((string) $tidy);
        }
    }

    // Attach the plugin as a listener
    $eventsManager->attach("view:afterRender", new TidyPlugin());

.. _this Github repository: https://github.com/bobthecow/mustache.php
.. _ajax request: http://api.jquery.com/jQuery.ajax/
.. _Tidy: http://www.php.net/manual/en/book.tidy.php
.. _md5: http://php.net/manual/en/function.md5.php
.. _PHP alternative site: https://github.com/ManaPHP/php-site
