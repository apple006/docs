Routing
=======

The router component allows you to define routes that are mapped to controllers or handlers that should receive
the request. A router simply parses a URI to determine this information. The router has two modes: MVC
mode and match-only mode. The first mode is ideal for working with MVC applications.

Defining Routes
---------------
:doc:`ManaPHP\\Mvc\\Router` provides advanced routing capabilities. In MVC mode,
you can define routes and map them to /modules/controllers/actions that you require. A route is defined as follows:

.. code-block:: php

    <?php
    
    //Create a route group, which contains some routes
    $group = new \ManaPHP\Mvc\Router\Group();

    //Add a route to the group
    $group->add("/admin/users/my-profile", array(
        "controller" => "users",
        "action" => "profile"
      ));

    //Add another route to the group
    $group->add("/admin/users/change-password", array(
        "controller" => "users",
        "action" => "changePassword"
      ));

    //Create the router
    $router = new \ManaPHP\Mvc\Router();
    $router->mount($group);

    $router->handle();

The first parameter of the :code:`add()` method is the pattern you want to match and, optionally, the second parameter is a set of paths.
In this case, if the URI is /admin/users/my-profile, then the "users" controller with its action "profile"
will be executed. It's important to remember that the router does not execute the controller and action, it only collects this
information to inform the correct component (ie. :doc:`ManaPHP\\Mvc\\Dispatcher`)
that this is the controller/action it should execute.

An application can have many paths and defining routes one by one can be a cumbersome task. In these cases we can
create more flexible routes:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router;

    // Create the router
    $router = new Router();

    // Define a route
    $router->add(
        "/admin/:controller/a/:action/:params",
        array(
            "controller" => 1,
            "action"     => 2,
            "params"     => 3
        )
    );

In the example above, we're using wildcards to make a route valid for many URIs. For example, by accessing the
following URL (/admin/users/a/delete/dave/301) would produce:

+------------+---------------+
| Controller | users         |
+------------+---------------+
| Action     | delete        |
+------------+---------------+
| Parameter  | dave          |
+------------+---------------+
| Parameter  | 301           |
+------------+---------------+

The :code:`add()` method receives a pattern that can optionally have predefined placeholders and regular expression
modifiers. All the routing patterns must start with a forward slash character (/). The regular expression syntax used
is the same as the `PCRE regular expressions`_. Note that, it is not necessary to add regular expression
delimiters. All route patterns are case-insensitive.

The second parameter defines how the matched parts should bind to the controller/action/parameters. Matching
parts are placeholders or subpatterns delimited by parentheses (round brackets). In the example given above, the
first subpattern matched (:code:`:controller`) is the controller part of the route, the second the action and so on.

These placeholders help writing regular expressions that are more readable for developers and easier
to understand. The following placeholders are supported:

+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| Placeholder          | Regular Expression          | Usage                                                                                                  |
+======================+=============================+========================================================================================================+
| :code:`/:module`     | :code:`/([a-zA-Z0-9\_\-]+)` | Matches a valid module name with alpha-numeric characters only                                         |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| :code:`/:controller` | :code:`/([a-zA-Z0-9\_\-]+)` | Matches a valid controller name with alpha-numeric characters only                                     |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| :code:`/:action`     | :code:`/([a-zA-Z0-9\_]+)`   | Matches a valid action name with alpha-numeric characters only                                         |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| :code:`/:params`     | :code:`(/.*)*`              | Matches a list of optional words separated by slashes. Only use this placeholder at the end of a route |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| :code:`/:namespace`  | :code:`/([a-zA-Z0-9\_\-]+)` | Matches a single level namespace name                                                                  |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| :code:`/:int`        | :code:`/([0-9]+)`           | Matches an integer parameter                                                                           |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+

Controller names are camelized, this means that characters (:code:`-`) and (:code:`_`) are removed and the next character
is uppercased. For instance, some_controller is converted to SomeController.

Since you can add many routes as you need using the :code:`add()` method, the order in which routes are added indicate
their relevance, latest routes added have more relevance than first added. Internally, all defined routes
are traversed in reverse order until :doc:`ManaPHP\\Mvc\\Router` finds the
one that matches the given URI and processes it, while ignoring the rest.

Parameters with Names
^^^^^^^^^^^^^^^^^^^^^
The example below demonstrates how to define names to route parameters:

.. code-block:: php

    <?php

    $router->add(
        "/news/([0-9]{4})/([0-9]{2})/([0-9]{2})/:params",
        array(
            "controller" => "posts",
            "action"     => "show",
            "year"       => 1, // ([0-9]{4})
            "month"      => 2, // ([0-9]{2})
            "day"        => 3, // ([0-9]{2})
            "params"     => 4  // :params
        )
    );

In the above example, the route doesn't define a "controller" or "action" part. These parts are replaced
with fixed values ("posts" and "show"). The user will not know the controller that is really dispatched
by the request. Inside the controller, those named parameters can be accessed as follows:

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
            // Get "year" parameter
            $year = $this->dispatcher->getParam("year");

            // Get "month" parameter
            $month = $this->dispatcher->getParam("month");

            // Get "day" parameter
            $day = $this->dispatcher->getParam("day");

            // ...
        }
    }

Note that the values of the parameters are obtained from the dispatcher. This happens because it is the
component that finally interacts with the drivers of your application. Moreover, there is also another
way to create named parameters as part of the pattern:

.. code-block:: php

    <?php

    $router->add(
        "/documentation/{chapter}/{name}.{type:[a-z]+}",
        array(
            "controller" => "documentation",
            "action"     => "show"
        )
    );

You can access their values in the same way as before:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Controller;

    class DocumentationController extends Controller
    {
        public function showAction()
        {
            // Get "name" parameter
            $name = $this->dispatcher->getParam("name");

            // Get "type" parameter
            $type = $this->dispatcher->getParam("type");

            // ...
        }
    }

Short Syntax
^^^^^^^^^^^^
If you don't like using an array to define the route paths, an alternative syntax is also available.
The following examples produce the same result:

.. code-block:: php

    <?php

    // Short form
    $router->add("/posts/{year:[0-9]+}/{title:[a-z\-]+}", "Posts::show");

    // Array form
    $router->add(
        "/posts/([0-9]+)/([a-z\-]+)",
        array(
           "controller" => "posts",
           "action"     => "show",
           "year"       => 1,
           "title"      => 2
        )
    );

Mixing Array and Short Syntax
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Array and short syntax can be mixed to define a route, in this case note that named parameters automatically
are added to the route paths according to the position on which they were defined:

.. code-block:: php

    <?php

    // First position must be skipped because it is used for
    // the named parameter 'country'
    $router->add('/news/{country:[a-z]{2}}/([a-z+])/([a-z\-+])',
        array(
            'section' => 2, // Positions start with 2
            'article' => 3
        )
    );

Routing to Modules
^^^^^^^^^^^^^^^^^^
You can define routes whose paths include modules. This is specially suitable to multi-module applications.
It's possible define a default route that includes a module wildcard:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router;

    $router = new Router(false);

    $router->add(
        '/:module/:controller/:action/:params',
        array(
            'module'     => 1,
            'controller' => 2,
            'action'     => 3,
            'params'     => 4
        )
    );

In this case, the route always must have the module name as part of the URL. For example, the following
URL: /admin/users/edit/sonny, will be processed as:

+------------+---------------+
| Module     | admin         |
+------------+---------------+
| Controller | users         |
+------------+---------------+
| Action     | edit          |
+------------+---------------+
| Parameter  | sonny         |
+------------+---------------+

Or you can bind specific routes to specific modules:

.. code-block:: php

    <?php

    $router->add(
        "/login",
        array(
            'module'     => 'backend',
            'controller' => 'login',
            'action'     => 'index'
        )
    );

    $router->add(
        "/products/:action",
        array(
            'module'     => 'frontend',
            'controller' => 'products',
            'action'     => 1
        )
    );

Or bind them to specific namespaces:

.. code-block:: php

    <?php

    $router->add(
        "/:namespace/login",
        array(
            'namespace'  => 1,
            'controller' => 'login',
            'action'     => 'index'
        )
    );

Namespaces/class names must be passed separated:

.. code-block:: php

    <?php

    $router->add(
        "/login",
        array(
            'namespace'  => 'Backend\Controllers',
            'controller' => 'login',
            'action'     => 'index'
        )
    );

HTTP Method Restrictions
^^^^^^^^^^^^^^^^^^^^^^^^
When you add a route using simply :code:`add()`, the route will be enabled for any HTTP method. Sometimes we can restrict a route to a specific method,
this is especially useful when creating RESTful applications:

.. code-block:: php

    <?php

    // This route only will be matched if the HTTP method is GET
    $router->addGet("/products/edit/{id}", "Products::edit");

    // This route only will be matched if the HTTP method is POST
    $router->addPost("/products/save", "Products::save");

    // This route will be matched if the HTTP method is POST or PUT
    $router->add("/products/update", "Products::update")->via(array("POST", "PUT"));

Using conversors
^^^^^^^^^^^^^^^^
Conversors allow you to freely transform the route's parameters before passing them to the dispatcher.
The following examples show how to use them:

.. code-block:: php

    <?php

    // The action name allows dashes, an action can be: /products/new-ipod-nano-4-generation
    $router
        ->add('/products/{slug:[a-z\-]+}', array(
            'controller' => 'products',
            'action'     => 'show'
        ))
        ->convert('slug', function ($slug) {
            // Transform the slug removing the dashes
            return str_replace('-', '', $slug);
        });

Another use case for conversors is binding a model into a route. This allows the model to be passed into the defined action directly:

.. code-block:: php

    <?php

    // This example works off the assumption that the ID is being used as parameter in the url: /products/4
    $router
        ->add('/products/{id}', array(
            'controller' => 'products',
            'action'     => 'show'
        ))
        ->convert('id', function ($id) {
            // Fetch the model
            return Product::findFirstById($id);
        });

Groups of Routes
^^^^^^^^^^^^^^^^
If a set of routes have common paths they can be grouped to easily maintain them:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router;
    use ManaPHP\Mvc\Router\Group as RouterGroup;

    $router = new Router();

    // Create a group with a common module and controller
    $blog = new RouterGroup(
        array(
            'module'     => 'blog',
            'controller' => 'index'
        )
    );

    // All the routes start with /blog
    $blog->setPrefix('/blog');

    // Add a route to the group
    $blog->add(
        '/save',
        array(
            'action' => 'save'
        )
    );

    // Add another route to the group
    $blog->add(
        '/edit/{id}',
        array(
            'action' => 'edit'
        )
    );

    // This route maps to a controller different than the default
    $blog->add(
        '/blog',
        array(
            'controller' => 'blog',
            'action'     => 'index'
        )
    );

    // Add the group to the router
    $router->mount($blog);

You can move groups of routes to separate files in order to improve the organization and code reusing in the application:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router\Group as RouterGroup;

    class BlogRoutes extends RouterGroup
    {
        public function initialize()
        {
            // Default paths
            $this->setPaths(
                array(
                    'module'    => 'blog',
                    'namespace' => 'Blog\Controllers'
                )
            );

            // All the routes start with /blog
            $this->setPrefix('/blog');

            // Add a route to the group
            $this->add(
                '/save',
                array(
                    'action' => 'save'
                )
            );

            // Add another route to the group
            $this->add(
                '/edit/{id}',
                array(
                    'action' => 'edit'
                )
            );

            // This route maps to a controller different than the default
            $this->add(
                '/blog',
                array(
                    'controller' => 'blog',
                    'action'     => 'index'
                )
            );
        }
    }

Then mount the group in the router:

.. code-block:: php

    <?php

    // Add the group to the router
    $router->mount(new BlogRoutes());

Matching Routes
---------------
A valid URI must be passed to the Router so that it can process it and find a matching route.
By default, the routing URI is taken from the :code:`$_GET['_url']` variable that is created by the rewrite engine
module. A couple of rewrite rules that work very well with ManaPHP are:

.. code-block:: apacheconf

    RewriteEngine On
    RewriteCond   %{REQUEST_FILENAME} !-d
    RewriteCond   %{REQUEST_FILENAME} !-f
    RewriteRule   ^((?s).*)$ index.php?_url=/$1 [QSA,L]

In this configuration, any requests to files or folders that don't exist will be sent to index.php.

The following example shows how to use this component in stand-alone mode:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router;

    // Creating a router
    $router = new Router();

    // Define routes here if any
    // ...

    // Taking URI from $_GET["_url"]
    $router->handle();

    // Or Setting the URI value directly
    $router->handle("/employees/edit/17");

    // Getting the processed controller
    echo $router->getControllerName();

    // Getting the processed action
    echo $router->getActionName();

    // Get the matched route
    $route = $router->getMatchedRoute();

Naming Routes
-------------
Each route that is added to the router is stored internally as a :doc:`ManaPHP\\Mvc\\Router\\Route` object.
That class encapsulates all the details of each route. For instance, we can give a name to a path to identify it uniquely in our application.
This is especially useful if you want to create URLs from it.

.. code-block:: php

    <?php

    $route = $router->add("/posts/{year}/{title}", "Posts::show");

    $route->setName("show-posts");

    // Or just

    $router->add("/posts/{year}/{title}", "Posts::show")->setName("show-posts");

Then, using for example the component :doc:`ManaPHP\\Mvc\\Url` we can build routes from its name:

.. code-block:: php

    <?php

    // Returns /posts/2012/ManaPHP-1-0-released
    echo $url->get(
        array(
            "for"   => "show-posts",
            "year"  => "2012",
            "title" => "ManaPHP-1-0-released"
        )
    );

Usage Examples
--------------
The following are examples of custom routes:

.. code-block:: php

    <?php

    // Matches "/system/admin/a/edit/7001"
    $router->add(
        "/system/:controller/a/:action/:params",
        array(
            "controller" => 1,
            "action"     => 2,
            "params"     => 3
        )
    );

    // Matches "/es/news"
    $router->add(
        "/([a-z]{2})/:controller",
        array(
            "controller" => 2,
            "action"     => "index",
            "language"   => 1
        )
    );

    // Matches "/es/news"
    $router->add(
        "/{language:[a-z]{2}}/:controller",
        array(
            "controller" => 2,
            "action"     => "index"
        )
    );

    // Matches "/admin/posts/edit/100"
    $router->add(
        "/admin/:controller/:action/:int",
        array(
            "controller" => 1,
            "action"     => 2,
            "id"         => 3
        )
    );

    // Matches "/posts/2015/02/some-cool-content"
    $router->add(
        "/posts/([0-9]{4})/([0-9]{2})/([a-z\-]+)",
        array(
            "controller" => "posts",
            "action"     => "show",
            "year"       => 1,
            "month"      => 2,
            "title"      => 4
        )
    );

    // Matches "/manual/en/translate.adapter.html"
    $router->add(
        "/manual/([a-z]{2})/([a-z\.]+)\.html",
        array(
            "controller" => "manual",
            "action"     => "show",
            "language"   => 1,
            "file"       => 2
        )
    );

    // Matches /feed/fr/le-robots-hot-news.atom
    $router->add(
        "/feed/{lang:[a-z]+}/{blog:[a-z\-]+}\.{type:[a-z\-]+}",
        "Feed::get"
    );

    // Matches /api/v1/users/peter.json
    $router->add(
        '/api/(v1|v2)/{method:[a-z]+}/{param:[a-z]+}\.(json|xml)',
        array(
            'controller' => 'api',
            'version'    => 1,
            'format'     => 4
        )
    );

.. highlights::

    Beware of characters allowed in regular expression for controllers and namespaces. As these
    become class names and in turn they're passed through the file system could be used by attackers to
    read unauthorized files. A safe regular expression is: :code:`/([a-zA-Z0-9\_\-]+)`

Default Behavior
----------------
:doc:`ManaPHP\\Mvc\\Router` has a default behavior that provides a very simple routing that
always expects a URI that matches the following pattern: /:controller/:action/:params

For example, for a URL like this *http://ManaPHPphp.com/documentation/show/about.html*, this router will translate it as follows:

+------------+---------------+
| Controller | documentation |
+------------+---------------+
| Action     | show          |
+------------+---------------+
| Parameter  | about.html    |
+------------+---------------+

If you don't want the router to have this behavior, you must create the router passing :code:`false` as the first parameter:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router;

    // Create the router without default routes
    $router = new Router(false);

Setting the default route
-------------------------
When your application is accessed without any route, the '/' route is used to determine what paths must be used to show the initial page
in your website/application:

.. code-block:: php

    <?php

    $router->add(
        "/",
        array(
            'controller' => 'index',
            'action'     => 'index'
        )
    );

Not Found Paths
---------------
If none of the routes specified in the router are matched, you can define a group of paths to be used in this scenario:

.. code-block:: php

    <?php

    // Set 404 paths
    $router->notFound(
        array(
            "controller" => "index",
            "action"     => "route404"
        )
    );

This is typically for an Error 404 page.

Setting default paths
---------------------
It's possible to define default values for the module, controller or action. When a route is missing any of
those paths they can be automatically filled by the router:

.. code-block:: php

    <?php

    // Setting a specific default
    $router->setDefaultModule('backend');
    $router->setDefaultNamespace('Backend\Controllers');
    $router->setDefaultController('index');
    $router->setDefaultAction('index');

    // Using an array
    $router->setDefaults(
        array(
            'controller' => 'index',
            'action'     => 'index'
        )
    );

Dealing with extra/trailing slashes
-----------------------------------
Sometimes a route could be accessed with extra/trailing slashes.
Those extra slashes would lead to produce a not-found status in the dispatcher.
You can set up the router to automatically remove the slashes from the end of handled route:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router;

    $router = new Router();

    // Remove trailing slashes automatically
    $router->removeExtraSlashes(true);

Or, you can modify specific routes to optionally accept trailing slashes:

.. code-block:: php

    <?php

    // The [/]{0,1} allows this route to have optionally have a trailing slash
    $router->add(
        '/{language:[a-z]{2}}/:controller[/]{0,1}',
        array(
            'controller' => 2,
            'action'     => 'index'
        )
    );

Match Callbacks
---------------
Sometimes, routes should only be matched if they meet specific conditions.
You can add arbitrary conditions to routes using the :code:`beforeMatch()` callback.
If this function return :code:`false`, the route will be treated as non-matched:

.. code-block:: php

    <?php

    $router->add('/login', array(
        'module'     => 'admin',
        'controller' => 'session'
    ))->beforeMatch(function ($uri, $route) {
        // Check if the request was made with Ajax
        if (isset($_SERVER['HTTP_X_REQUESTED_WITH'])
            && $_SERVER['HTTP_X_REQUESTED_WITH'] == 'XMLHttpRequest') {
            return false;
        }
        return true;
    });

You can re-use these extra conditions in classes:

.. code-block:: php

    <?php

    class AjaxFilter
    {
        public function check()
        {
            return $_SERVER['HTTP_X_REQUESTED_WITH'] == 'XMLHttpRequest';
        }
    }

And use this class instead of the anonymous function:

.. code-block:: php

    <?php

    $router->add('/get/info/{id}', array(
        'controller' => 'products',
        'action'     => 'info'
    ))->beforeMatch(array(new AjaxFilter(), 'check'));

Hostname Constraints
--------------------
The router allows you to set hostname constraints, this means that specific routes or a group of routes can be restricted
to only match if the route also meets the hostname constraint:

.. code-block:: php

    <?php

    $router->add('/login', array(
        'module'     => 'admin',
        'controller' => 'session',
        'action'     => 'login'
    ))->setHostName('admin.company.com');

The hostname can also be passed as a regular expressions:

.. code-block:: php

    <?php

    $router->add('/login', array(
        'module'     => 'admin',
        'controller' => 'session',
        'action'     => 'login'
    ))->setHostName('([a-z]+).company.com');

In groups of routes you can set up a hostname constraint that apply for every route in the group:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router\Group as RouterGroup;

    // Create a group with a common module and controller
    $blog = new RouterGroup(
        array(
            'module'     => 'blog',
            'controller' => 'posts'
        )
    );

    // Hostname restriction
    $blog->setHostName('blog.mycompany.com');

    // All the routes start with /blog
    $blog->setPrefix('/blog');

    // Default route
    $blog->add(
        '/',
        array(
            'action' => 'index'
        )
    );

    // Add a route to the group
    $blog->add(
        '/save',
        array(
            'action' => 'save'
        )
    );

    // Add another route to the group
    $blog->add(
        '/edit/{id}',
        array(
            'action' => 'edit'
        )
    );

    // Add the group to the router
    $router->mount($blog);

URI Sources
-----------
By default the URI information is obtained from the :code:`$_GET['_url']` variable, this is passed by the Rewrite-Engine to
ManaPHP, you can also use :code:`$_SERVER['REQUEST_URI']` if required:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router;

    // ...

    $router->setUriSource(Router::URI_SOURCE_GET_URL); // Use $_GET['_url'] (default)
    $router->setUriSource(Router::URI_SOURCE_SERVER_REQUEST_URI); // Use $_SERVER['REQUEST_URI']

Or you can manually pass a URI to the :code:`handle()` method:

.. code-block:: php

    <?php

    $router->handle('/some/route/to/handle');

Testing your routes
-------------------
Since this component has no dependencies, you can create a file as shown below to test your routes:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router;

    // These routes simulate real URIs
    $testRoutes = array(
        '/',
        '/index',
        '/index/index',
        '/index/test',
        '/products',
        '/products/index/',
        '/products/show/101',
    );

    $router = new Router();

    // Add here your custom routes
    // ...

    // Testing each route
    foreach ($testRoutes as $testRoute) {

        // Handle the route
        $router->handle($testRoute);

        echo 'Testing ', $testRoute, '<br>';

        // Check if some route was matched
        if ($router->wasMatched()) {
            echo 'Controller: ', $router->getControllerName(), '<br>';
            echo 'Action: ', $router->getActionName(), '<br>';
        } else {
            echo 'The route wasn\'t matched by any route<br>';
        }

        echo '<br>';
    }

Registering Router instance
---------------------------
You can register router during service registration with ManaPHP dependency injector to make it available inside the controllers.

You need to add code below in your bootstrap file (for example index.php or app/config/services.php if you use `ManaPHP Developer Tools`_)

.. code-block:: php

    <?php

    /**
     * Add routing capabilities
     */
    $di->set(
        'router',
        function () {
            require __DIR__.'/../app/config/routes.php';

            return $router;
        }
    );

You need to create app/config/routes.php and add router initialization code, for example:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Router;

    $router = new Router();

    $router->add(
        "/login",
        array(
            'controller' => 'login',
            'action'     => 'index'
        )
    );

    $router->add(
        "/products/:action",
        array(
            'controller' => 'products',
            'action'     => 1
        )
    );

    return $router;

Implementing your own Router
----------------------------
The :doc:`ManaPHP\\Mvc\\RouterInterface` interface must be implemented to create your own router replacing
the one provided by ManaPHP.

.. _PCRE regular expressions: http://www.php.net/manual/en/book.pcre.php
