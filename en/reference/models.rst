Working with Models
===================

A model represents the information (data) of the application and the rules to manipulate that data. Models are primarily used for managing
the rules of interaction with a corresponding database table. In most cases, each table in your database will correspond to one model in
your application. The bulk of your application's business logic will be concentrated in the models.

:doc:`ManaPHP\\Mvc\\Model` is the base for all models in a ManaPHP application. It provides database independence, basic
CRUD functionality, advanced finding capabilities, among other services.
:doc:`ManaPHP\\Mvc\\Model` avoids the need of having to use SQL statements because it translates
methods dynamically to the respective database engine operations.

.. highlights::

    Models are intended to work on a database high layer of abstraction. If you need to work with databases at a lower level check out the
    :doc:`ManaPHP\\Db` component documentation.

Creating Models
---------------
A model is a class that extends from :doc:`ManaPHP\\Mvc\\Model`. It must be placed in the models directory. A model
file must contain a single class; its class name should be in camel case notation:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {

    }

The above example shows the implementation of the "City" model. Note that the class City inherits from :doc:`ManaPHP\\Mvc\\Model`.
This component provides a great deal of functionality to models that inherit it, including basic database
CRUD (Create, Read, Update, Delete) operations, as well as sophisticated search support.

.. highlights::

    If you're using PHP 5.4/5.5 it is recommended you declare each column that makes part of the model in order to save
    memory and reduce the memory allocation.

By default, the model "City" will refer to the table "city". If you want to manually specify another name for the mapping table,
you can use the :code:`getSource()` method:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public function getSource()
        {
            return "the_city";
        }
    }

The model City now maps to "the_city" table. The :code:`initialize()` method aids in setting up the model with a custom behavior i.e. a different table.
The :code:`initialize()` method is only called once during the request.

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public function initialize()
        {
            $this->setSource("the_city");
        }
    }

The :code:`initialize()` method is only called once during the request, it's intended to perform initializations that apply for
all instances of the model created within the application. If you want to perform initialization tasks for every instance
created you can 'onConstruct':

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public function onConstruct()
        {
            // ...
        }
    }

Public properties
^^^^^^^^^^^^^^^^^
Models can be implemented with properties of public scope, meaning that each property can be read/updated
from any part of the code that has instantiated that model class without any restrictions:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public $city_id;

        public $city_name;

        public $country_id;
    }

Public properties provide less complexity in development.

Models in Namespaces
^^^^^^^^^^^^^^^^^^^^
Namespaces can be used to avoid class name collision. The mapped table is taken from the class name, in this case 'City':

.. code-block:: php

    <?php

    namespace Application\Home\Models;

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        // ...
    }

Understanding Records To Objects
--------------------------------
Every instance of a model represents a row in the table. You can easily access record data by reading object properties. For example,
for a table "city" with the records:

.. code-block:: bash

    mysql> select * from city;

    +---------+------------+------------+
    | city_id | city_name  | country_id |
    +---------+------------+------------+
    |  1      | Rob        | 76         |
    |  2      | Aden       | 25         |
    |  3      | Ado        | 16         |
    +---------+------------+------------+

    3 rows in set (0.00 sec)

You could find a certain record by its primary key and then print its name:

.. code-block:: php

    <?php

    // Find record with city_id = 3
    $city = City::findFirst(3);

    // Prints "Ado"
    echo $city->city_name;

Once the record is in memory, you can make modifications to its data and then save changes:

.. code-block:: php

    <?php

    $city = City::findFirst(3);
    $city->city_name = "Beijing";
    $city->save();

As you can see, there is no need to use raw SQL statements. :doc:`ManaPHP\\Mvc\\Model` provides high database
abstraction for web applications.

Finding Records
---------------
:doc:`ManaPHP\\Mvc\\Model` also offers several methods for querying records. The following examples will show you
how to query one or more records from a model:

.. code-block:: php

    <?php

    // How many cities are there?
    $cities = City::find();
    echo "There are ", count($cities), "\n";

    // How many cities of country_id is equal to 1 are there?
    $cities = City::find(['country_id'=>1]);
    echo "There are ", count($cities), "\n";

    // Get and print cities where country_id is equal to 1 ordered by city_name
    $cities = City::find(
        array(
            ['country_id'=>1],
            "order" => "city_name ASC"
        )
    );
    foreach ($cities as $city) {
        echo $city->city_name, "\n";
    }

    // Get first 100 cities ordered by city_name ASC
    $cities = City::find(
        array(
            "",//any record
            "order" => "city_name",
            "limit" => 100
        )
    );
    foreach ($cities as $city) {
       echo $city->city_name, "\n";
    }

.. highlights::

    If you want find record by external data (such as user input) or variable data you must use `Binding Parameters`_.

You could also use the :code:`findFirst()` method to get only the first record matching the given criteria:

.. code-block:: php

    <?php

    // What's the first city in city table?
    $city = City::findFirst();
    echo "The city name is ", $city->city_name, "\n";

    // What's the first city of country_id is equal to 1 in city table?
    $city = City::findFirst(['country_id'=>1]);
    echo "The first city name is ", $city->city_name, "\n";

    // Get first city ordered by city_name
    $city = City::findFirst(
        array(
            '',//any record
            "order" => "city_name"
        )
    );
    echo "The first city name is ", $city->city_name, "\n";

Both :code:`find()` and :code:`findFirst()` methods accept an associative array specifying the search criteria:

.. code-block:: php

    <?php

    $city = City::findFirst(
        array(
            ['country_id' => 10],
            "order" => "name DESC",
            "limit" => 30
        )
    );

    $city = City::find(
        array(
            "conditions" => "country_id = :city_id",
            "bind"       => array('city_id' => "10")
        )
    );

The available query options are:

+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------+
| Parameter   | Description                                                                                                                                                                                                                          | Example                                                                         |
+=============+======================================================================================================================================================================================================================================+=================================================================================+
| conditions  | Search conditions for the find operation. Is used to extract only those records that fulfill a specified criterion. By default :doc:`ManaPHP\\Mvc\\Model` assumes the first parameter are the conditions. | :code:`"conditions" => "name LIKE 'steve%'"`                                    |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------+
| columns     | Return specific columns instead of the full columns in the model. When using this option an incomplete object is returned                                                                                                            | :code:`"columns" => "id, name"`                                                 |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------+
| bind        | Bind is used together with options, by replacing placeholders and escaping values thus increasing security                                                                                                                           | :code:`"bind" => array("status" => "A", "type" => "some-time")`                 |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------+
| order       | Is used to sort the resultset. Use one or more fields separated by commas.                                                                                                                                                           | :code:`"order" => "name DESC, status"`                                          |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------+
| limit       | Limit the results of the query to results to certain range                                                                                                                                                                           | :code:`"limit" => 10`                                                           |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------+
| offset      | Offset the results of the query by a certain amount                                                                                                                                                                                  | :code:`"offset" => 5`                                                           |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------+
| group       | Allows to collect data across multiple records and group the results by one or more columns                                                                                                                                          | :code:`"group" => "name, status"`                                               |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------+
| cache       | Cache the resultset, reducing the continuous access to the relational system                                                                                                                                                         | :code:`"cache" => array("lifetime" => 3600, "key" => "my-find-key")`            |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------+

If you prefer, there is also available a way to create queries in an object-oriented way, :doc:`\ManaPHP\Mvc\Model\Query\Builder `instead of using an array of parameters,which
is friendly with IDE auto completes :

.. code-block:: php
    <?php

    $builder=new ManaPHP\Mvc\Model\Query\Builder();
    $builder->columns('*')
            ->addFrom(City::class)
            ->where(['city_id >=:city_id',['city_id'=>10])
            ->orderBy('city_id DESC')
            ->getQuery()->execute();

All the :doc:`ManaPHP\\Mvc\\Model` queries are internally handled by :doc:`\\ManaPHP\\Mvc\\Model\\Query\\Builder` object.

Model Resultsets
^^^^^^^^^^^^^^^^
While :code:`findFirst()` returns directly an instance of the called class (when there is data to be returned), the :code:`find()` method returns an
array, everyone is an instance of the called class (when there is data to be returned).

.. code-block:: php

    <?php

    // Get all cities
    $cities = City::find();

    // Traversing with a foreach
    foreach ($cities as $city) {
        echo $city->city_name, "\n";
    }

    // Count the resultset
    echo count($cities);

    // Access a city by its position in the resultset
    $city = $cities[5];

    // Check if there is a record in certain position
    if (isset($cities[3])) {
       $city = $cities[3];
    }

Binding Parameters
^^^^^^^^^^^^^^^^^^
Bound parameters are also supported in :doc:`ManaPHP\\Mvc\\Model`. You are encouraged to use
this methodology so as to eliminate the possibility of your code being subject to SQL injection attacks.
**Only string placeholders** are supported. Binding parameters can simply be achieved as follows:

.. code-block:: php

    <?php

    // Query cities binding parameters with string placeholders
    $conditions = "city_name = :name";

    // Parameters whose keys are the same as placeholders
    $parameters = array(
        "city_name" => "Rob",
    );

    // Perform the query
    $cities = City::find(
        array(
            $conditions,
            "bind" => $parameters
        )
    );

Strings are automatically escaped using PDO_. This function takes into account the connection charset, so its recommended to define
the correct charset in the connection parameters or in the database configuration, as a wrong charset will produce undesired effects
when storing or retrieving data.

.. highlights::

    Bound parameters are available for all query methods such as :code:`find()` and :code:`findFirst()` but also the calculation
    methods like :code:`count()`, :code:`sum()`, :code:`average()` etc.

Initializing/Preparing fetched records
--------------------------------------
May be the case that after obtaining a record from the database is necessary to initialise the data before
being used by the rest of the application. You can implement the method 'afterFetch' in a model, this event
will be executed just after create the instance and assign the data to it:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public $id;

        public $name;

        public $status;

        public function beforeSave()
        {
            // Convert the array into a string
            $this->status = join(',', $this->status);
        }

        public function afterFetch()
        {
            // Convert the string to an array
            $this->status = explode(',', $this->status);
        }
        
        public function afterSave()
        {
            // Convert the string to an array
            $this->status = explode(',', $this->status);
        }
    }

Generating Calculations
-----------------------
Calculations (or aggregations) are helpers for commonly used functions of database systems such as COUNT, SUM, MAX, MIN or AVG.
:doc:`ManaPHP\\Mvc\\Model` allows to use these functions directly from the exposed methods.

Count examples:

.. code-block:: php

    <?php

    // How many employees are?
    $rowcount = Employees::count();

    // How many different areas are assigned to employees?
    $rowcount = Employees::count(
        array(
            "distinct" => "area"
        )
    );

    // How many employees are in the Testing area?
    $rowcount = Employees::count(
        "area = 'Testing'"
    );

    // Count employees grouping results by their area
    $group = Employees::count(
        array(
            "group" => "area"
        )
    );
    foreach ($group as $row) {
       echo "There are ", $row->row_count, " in ", $row->area;
    }

    // Count employees grouping by their area and ordering the result by count
    $group = Employees::count(
        array(
            "group" => "area",
            "order" => "row_count"
        )
    );

    // Avoid SQL injections using bound parameters
    $group = Employees::count(
        array(
            "type > ?0",
            "bind" => array($type)
        )
    );

Sum examples:

.. code-block:: php

    <?php

    // How much are the salaries of all employees?
    $total = Employees::sum("salary");

    // How much are the salaries of all employees in the Sales area?
    $total = Employees::sum('salary',['area'=> 'Sales']);

    // Generate a grouping of the salaries of each area
    $group = Employees::sum('salary',['','group'  => "area"]);
    foreach ($group as $row) {
       echo "The sum of salaries of the ", $row->area, " is ", $row->summary;
    }

    // Generate a grouping of the salaries of each area ordering
    // salaries from higher to lower
    $group = Employees::sum('salary',['',"group" => "area","order" => "summary DESC"]);

    // Avoid SQL injections using bound parameters
    $group = Employees::sum('salary'
        array(
            "conditions" => "area > :area",
            "bind"       => array('area'=>0)
        )
    );

Average examples:

.. code-block:: php

    <?php

    // What is the average salary for all employees?
    $average = Employees::average('salary');

    // What is the average salary for the Sales's area employees?
    $average = Employees::average('salary',['area' => 'Sales']);

Max/Min examples:

.. code-block:: php

    <?php

    // What is the oldest age of all employees?
    $age = Employees::maximum('age');

    // What is the oldest of employees from the Sales area?
    $age = Employees::maximum('age',['area' => 'Sales']);

    // What is the lowest salary of all employees?
    $salary = Employees::minimum('salary');

Creating Updating/Records
-------------------------
The method :code:`ManaPHP\Mvc\Model::save()` allows you to create/update records according to whether they already exist in the table
associated with a model. The save method is called internally by the create and update methods of :doc:`ManaPHP\\Mvc\\Model`.
For this to work as expected it is necessary to have properly defined a primary key in the entity to determine whether a record
should be updated or created.

Also the method executes associated validators and events that are defined in the model:

.. code-block:: php

    <?php

    $city = new City();
    $city->city_name = "beijing";
    $city->country_id = 200;

    try{
        $city->save();

        echo "Great, a new city was saved successfully!";
    }catch(\Exception $e){
        echo "Umh, We can't store city right now: \n";
    }

An array could be passed to "save" to avoid assign every column manually:

.. code-block:: php

    <?php

    $city = new City();

    $city->save(
        array(
            "city_name" => "beijing",
            "country_id" => 200,
        )
    );

Values assigned directly or via the array of attributes are escaped/sanitized according to the related attribute data type. So you can pass
an insecure array without worrying about possible SQL injections:

.. code-block:: php

    <?php

    $city = new City();
    $city->save($_POST);

.. highlights::

    Without precautions mass assignment could allow attackers to set any database column's value. Only use this feature
    if you want to permit a user to insert/update every column in the model, even if those fields are not in the submitted
    form.

You can set an additional parameter in 'save' to set a whitelist of fields that only must taken into account when doing
the mass assignment:

.. code-block:: php

    <?php

    $city = new City();

    $city->save(
        $_POST,
        array(
            'city_name',
            'country_id'
        )
    );

Create/Update with Confidence
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When an application has a lot of competition, we could be expecting create a record but it is actually updated. This
could happen if we use :code:`ManaPHP\Mvc\Model::save()` to persist the records in the database. If we want to be absolutely
sure that a record is created or updated, we can change the :code:`save()` call with :code:`create()` or :code:`update()`:

.. code-block:: php

    <?php

    $city = new City();
    $city->city_name = "Beijing";
    $city->country_id = 1952;

    // This record only must be created
    try{
        $city->create();

        echo "Great, a new city was saved successfully!";
    }catch(\Exception $e){
        echo "Umh, We can't store city right now: \n";
    }

These methods "create" and "update" also accept an array of values as parameter.

Auto-generated identity columns
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Some models may have identity columns. These columns usually are the primary key of the mapped table. :doc:`ManaPHP\\Mvc\\Model`
can recognize the identity column omitting it in the generated SQL INSERT, so the database system can generate an auto-generated value for it.
Always after creating a record, the identity field will be registered with the value generated in the database system for it:

.. code-block:: php

    <?php

    $city->save();

    echo "The generated id is: ", $city->city_id;

:doc:`ManaPHP\\Mvc\\Model` is able to recognize the identity column. Depending on the database system, those columns may be
serial columns like in PostgreSQL or auto_increment columns in the case of MySQL.

PostgreSQL uses sequences to generate auto-numeric values, by default, ManaPHP tries to obtain the generated value from the sequence "table_field_seq",
for example: city_id_seq, if that sequence has a different name, the method "getSequenceName" needs to be implemented:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public function getSequenceName()
        {
            return "city_sequence_name";
        }
    }

Events and Events Manager
^^^^^^^^^^^^^^^^^^^^^^^^^
Models allow you to implement events that will be thrown when performing an insert/update/delete. They help define business rules for a
certain model. The following are the events supported by :doc:`ManaPHP\\Mvc\\Model` and their order of execution:

+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting/Updating | beforeSave               | YES                   | Runs before the required operation over the database system                                                                       |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Updating           | beforeUpdate             | YES                   | Runs before the required operation over the database system only when an updating operation is being made                         |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting          | beforeCreate             | YES                   | Runs before the required operation over the database system only when an inserting operation is being made                        |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Updating           | afterUpdate              | NO                    | Runs after the required operation over the database system only when an updating operation is being made                          |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting          | afterCreate              | NO                    | Runs after the required operation over the database system only when an inserting operation is being made                         |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| Inserting/Updating | afterSave                | NO                    | Runs after the required operation over the database system                                                                        |
+--------------------+--------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+

Implementing Events in the Model's class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The easier way to make a model react to events is implement a method with the same name of the event in the model's class:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public function beforeCreate()
        {
            echo "This is executed before creating a City!";
        }
    }

Events can be useful to assign values before performing an operation, for example:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class Product extends Model
    {
        public function beforeCreate()
        {
            // Set the creation date
            $this->created_at = date('Y-m-d H:i:s');
        }

        public function beforeUpdate()
        {
            // Set the modification date
            $this->modified_in = date('Y-m-d H:i:s');
        }
    }

Implementing a Business Rule
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When an insert, update or delete is executed, the model verifies if there are any methods with the names of
the events listed in the table above.

We recommend that validation methods are declared protected to prevent that business logic implementation
from being exposed publicly.

The following example implements an event that validates the year cannot be smaller than 0 on update or insert:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public function beforeSave()
        {
            if ($this->country_id < 0) {
                echo "country_id cannot be smaller than zero!";
                return false;
            }
        }
    }

Some events return false as an indication to stop the current operation. If an event doesn't return anything, :doc:`ManaPHP\\Mvc\\Model` will assume a true value.

Avoiding SQL injections
^^^^^^^^^^^^^^^^^^^^^^^
Every value assigned to a model attribute is escaped depending of its data type. A developer doesn't need to escape manually
each value before storing it on the database. ManaPHP uses internally the `bound parameters <http://php.net/manual/en/pdostatement.bindparam.php>`_
capability provided by PDO to automatically escape every value to be stored in the database.

.. code-block:: bash

    mysql> desc product;
    +------------------+------------------+------+-----+---------+----------------+
    | Field            | Type             | Null | Key | Default | Extra          |
    +------------------+------------------+------+-----+---------+----------------+
    | id               | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
    | type_id          | int(10) unsigned | NO   | MUL | NULL    |                |
    | name             | varchar(70)      | NO   |     | NULL    |                |
    | price            | decimal(16,2)    | NO   |     | NULL    |                |
    | active           | char(1)          | YES  |     | NULL    |                |
    +------------------+------------------+------+-----+---------+----------------+
    5 rows in set (0.00 sec)

If we use just PDO to store a record in a secure way, we need to write the following code:

.. code-block:: php

    <?php

    $name           = 'Artichoke';
    $price          = 10.5;
    $active         = 'Y';
    $type_id        = 1;

    $sql = 'INSERT INTO product VALUES (null, :type_id, :name, :price, :active)';
    $sth = $dbh->prepare($sql);

    $sth->bindParam(':type_id', $type_id, PDO::PARAM_INT);
    $sth->bindParam(':name', $name, PDO::PARAM_STR, 70);
    $sth->bindParam(':price', doubleval($price));
    $sth->bindParam(':active', $active, PDO::PARAM_STR, 1);

    $sth->execute();

The good news is that ManaPHP do this for you automatically:

.. code-block:: php

    <?php

    $product                   = new Product();
    $product->type_id          = 1;
    $product->name             = 'Artichoke';
    $product->price            = 10.5;
    $product->active           = 'Y';

    $product->create();

Deleting Records
----------------
The method :code:`ManaPHP\Mvc\Model::delete()` allows to delete a record. You can use it as follows:

.. code-block:: php

    <?php

    $city = City::findFirst(11);

    if ($city != false) {
        if ($city->delete() == false) {
            echo "Sorry, we can't delete the city right now: \n";
        } else {
            echo "The city was deleted successfully!";
        }
    }

You can also delete many records by traversing a resultset with a foreach:

.. code-block:: php

    <?php

    foreach (City::find(['country_id'=>1]) as $city) {
        if ($city->delete() == false) {
            echo "Sorry, we can't delete the city right now: \n";
        } else {
            echo "The city was deleted successfully!";
        }
    }

The following events are available to define custom business rules that can be executed when a delete operation is
performed:

+-----------+--------------+---------------------+------------------------------------------+
| Operation | Name         | Can stop operation? | Explanation                              |
+===========+==============+=====================+==========================================+
| Deleting  | beforeDelete | YES                 | Runs before the delete operation is made |
+-----------+--------------+---------------------+------------------------------------------+
| Deleting  | afterDelete  | NO                  | Runs after the delete operation was made |
+-----------+--------------+---------------------+------------------------------------------+

With the above events can also define business rules in the models:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public function beforeDelete()
        {
            if ($this->status == 'A') {
                echo "The city is active, it can't be deleted";

                return false;
            }

            return true;
        }
    }

Record Snapshots
----------------
Specific models could be set to maintain a record snapshot when they're queried. You can use this feature to implement auditing or just to know what
fields are changed according to the data queried from the persistence:

The application consumes a bit more of memory to keep track of the original values obtained from the persistence. you can check what fields changed:

.. code-block:: php

    <?php

    // Get a record from the database
    $city = City::findFirst();

    // Change a column
    $city->city_name = 'Other name';

    var_dump($city->getChangedFields()); // ['city_name']
    var_dump($city->hasChanged('city_name')); // true
    var_dump($city->hasChanged('city_id')); // false

Setting multiple databases
--------------------------
In ManaPHP, all models can belong to the same database connection or have an individual one. Actually, when
:doc:`ManaPHP\\Mvc\\Model` needs to connect to the database it requests the "db" service
in the application's services container. You can overwrite this service setting it in the initialize method:

.. code-block:: php

    <?php

    use ManaPHP\Db\Adapter\Pdo\Mysql as MysqlPdo;
    use ManaPHP\Db\Adapter\Pdo\PostgreSQL as PostgreSQLPdo;

    // This service returns a MySQL database
    $di->set('dbMysql', function () {
        return new MysqlPdo(
            array(
                "host"     => "localhost",
                "username" => "root",
                "password" => "secret",
                "dbname"   => "invo"
            )
        );
    });

    // This service returns a PostgreSQL database
    $di->set('dbPostgres', function () {
        return new PostgreSQLPdo(
            array(
                "host"     => "localhost",
                "username" => "postgres",
                "password" => "",
                "dbname"   => "invo"
            )
        );
    });

Then, in the initialize method, we define the connection service for the model:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public function initialize()
        {
            $this->setConnectionService('dbPostgres');
        }
    }

But ManaPHP offers you more flexibility, you can define the connection that must be used to 'read' and for 'write'. This is specially useful
to balance the load to your databases implementing a master-slave architecture:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public function initialize()
        {
            $this->setReadConnectionService('dbSlave');
            $this->setWriteConnectionService('dbMaster');
        }
    }

Logging Low-Level SQL Statements
--------------------------------
When using high-level abstraction components such as :doc:`ManaPHP\\Mvc\\Model` to access a database, it is
difficult to understand which statements are finally sent to the database system. :doc:`ManaPHP\\Mvc\\Model`
is supported internally by :doc:`ManaPHP\\Db`. :doc:`ManaPHP\\Log\\Logger` interacts with :doc:`ManaPHP\\Db`,
providing logging capabilities on the database abstraction layer, thus allowing us to log SQL statements as they happen.

.. code-block:: php

    <?php

    use ManaPHP\Logger;
    use ManaPHP\Events\Manager;
    use ManaPHP\Db\Adapter\Pdo\Mysql as Connection;

    $di->set('db', function () use($logger) {

        $logger = new \ManaPHP\Log\Logger();
        $logger->addAdapter(new \ManaPHP\Log\Adapter\File('app/logs/debug.log'));

        $connection = new Connection(
            array(
                "host"     => "localhost",
                "username" => "root",
                "password" => "secret",
                "dbname"   => "invo"
            )
        );
        $connection->attachEvent('db:beforeQuery', function ($event, DbInterface $source, $data) use ($logger) {
                    $logger->debug('SQL: ' . $source->getSQLStatement());
                });

        return $connection;
    });

As models access the default database connection, all SQL statements that are sent to the database system will be logged in the file:

.. code-block:: php

    <?php

    $city = new City();
    $city->city_name = "Robby";
    $city->country_id=100;
    try{
        $city->save()
    }catch(\Exception $e){
        echo "Cannot save city";
    }

As above, the file *app/logs/db.log* will contain something like this:

.. code-block:: irc

    [Mon, 30 Apr 12 13:47:18 -0500][DEBUG] SQL: INSERT INTO city
    (city_name, country_id) VALUES (:city_name, :country_id)

Injecting services into Models
------------------------------
You may be required to access the application services within a model, the following example explains how to do that:

.. code-block:: php

    <?php

    use ManaPHP\Mvc\Model;

    class City extends Model
    {
        public function notSaved()
        {
            $this->logger->debug('not saved');
        }
    }

Stand-Alone component
---------------------
Using :doc:`ManaPHP\\Mvc\\Model <models>` in a stand-alone mode can be demonstrated below:

.. code-block:: php

    <?php

    use ManaPHP\Di;
    use ManaPHP\Mvc\Model;
    use ManaPHP\Mvc\Model\Manager as ModelsManager;
    use ManaPHP\Db\Adapter\Pdo\Sqlite as Connection;
    use ManaPHP\Mvc\Model\Metadata\Memory as MetaData;

    $di = new Di();

    // Setup a connection
    $di->set(
        'db',
        new Connection(
            array(
                "dbname" => "sample.db"
            )
        )
    );

    // Set a models manager
    $di->set('modelsManager', new ModelsManager());

    // Use the memory meta-data adapter or other
    $di->set('modelsMetadata', new MetaData());

    // Create a model
    class City extends Model
    {

    }

    // Use the model
    echo City::count();

.. _PDO: http://php.net/manual/en/pdo.prepared-statements.php
.. _date: http://php.net/manual/en/function.date.php
.. _time: http://php.net/manual/en/function.time.php
.. _Traits: http://php.net/manual/en/language.oop5.traits.php
