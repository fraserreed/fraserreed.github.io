---
layout: post
title:  "Database Testing with PHPUnit"
summary: 
date:   2016-04-06 13:28:00
categories: phpunit
comments: true
---

In this post, you will learn how to set up PHPUnit to test the database layer of your application.  Unit tests are usually
focused on testing object models, but they can also be used to test interactions with the database to prevent
regressions in data persistence.  If you have a complex application that involves a database, it is very important to 
validate that it is working properly so you can be more confident in your deploys.

<!--more-->

The examples used in this post are based on a sample "library" application.  You can find all source code for the app 
[on Github](https://github.com/fraserreed/blog-samples/tree/master/database-testing).  The sample has an accompanying
Vagrantfile for creating an ubuntu VM with the sample database initialized, and uses [Phinx](https://phinx.org/) for 
managing the database migration.  Follow the README for steps to initialize.


# Application Summary

The application is a basic example of a library system.  There are two tables: `authors` and `books`.

{% highlight sql %}
CREATE TABLE `authors` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

CREATE TABLE `books` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `isbn` varchar(16) NOT NULL,
  `author_id` int(11) NOT NULL,
  `title` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
{% endhighlight %}

Each record in the `books` table represents a book in the library.  Each book has an ISBN number, a title and an author, 
which is stored using an `author_id`.  The `author_id` in the `books` table is a foreign key to the `id` in the `authors` 
table.

In the sample database `library`, there are four books:

{% highlight html linenos %}
A Short History of Nearly Everything - Bill Bryson - 9780767908184
The Life and Times of the Thunderbolt Kid - Bill Bryson - 9780767919371
The Road to Little Dribbling - Bill Bryson - 9780385539289
The Kite Runner - Khaled Hosseini - 9781594480003
{% endhighlight %}

# Code Structure

The code structure of the application is shown below. 

![database testing directory structure](/css/images/post-assets/database-testing-directory.png)

The source code is located in the `src` directory, and the corresponding tests are located in the `tests` directory.
In a fully tested application there should be a corresponding `tests/Unit` folder at the same level as the `tests/Persistence` folder,
but this sample application only covers the `Persistence` (database) testing requirements so the unit tests have been left out.

The `BookMapper` class is the database mapper class that queries the database table and maps the resulting records
into `Book` objects.  There are 5 functions in the class that will be tested:

* `fetchAll` 
  * fetches all books in the `books` table, with author name from the `authors` table
* `fetchByISBN`
  * fetches zero or one book in the `books` table, matching the provided ISBN number
* `insert`
  * creates a new record in the `books` table
* `update`
  * updates a record in the `books` table
* `delete`
  * deletes a record in the `books` table
 
 Each of these functions will be tested using the `BookMapperTest.php` class.

# Setting Up The Tests

The first thing to do when setting up the tests is to install [PHPUnit](https://phpunit.de/).  The best way to do this is
using [Composer](https://getcomposer.org/).

Ensure that both `phpunit/phpunit` and `phpunit/dbunit` are included in the `require-dev` section of the `composer.json` 
file.

{% highlight json %}
{
  ..

  "require-dev":{
    "phpunit/phpunit": "~4.8",
    "phpunit/dbunit": "~2"
  },

  ..
}
{% endhighlight %}

Next create a file `phpunit.xml.dist` at the root of the project to define the bootstrap for the project, the database
connection details for the tests, and any specific test suites that you require.  In the case of this sample, there 
is only a `persistence` suite:

{% highlight xml %}
<phpunit bootstrap="./tests/bootstrap.php"
         verbose="false"
         convertErrorsToExceptions="true"
         convertNoticesToExceptions="true"
         convertWarningsToExceptions="true">
         
    <php>
        <var name="DB_HOST" value="10.0.0.20" />
        <var name="DB_USER" value="root" />
        <var name="DB_PASSWD" value="tutorial" />
        <var name="DB_DBNAME" value="library" />
    </php>

    <testsuites>
        <testsuite name="persistence">
            <directory>./tests/Persistence</directory>
        </testsuite>
    </testsuites>
</phpunit>
{% endhighlight %}

Finally create a `bootstrap.php` file at the root of the `tests` directory to initialize the tests for phpunit, and create 
an empty `tests/Persistence` directory.

{% highlight php %}
<?php
error_reporting( E_ALL | E_STRICT );
// Ensure that composer has installed all dependencies
if( !file_exists( dirname( __DIR__ ) . '/composer.lock' ) )
{
    die( "Dependencies must be installed using composer:\n\nphp composer.phar install --dev\n\n"
        . "See http://getcomposer.org for help with installing composer\n" );
}
// Include the composer autoloader
$autoloader = require dirname( __DIR__ ) . '/vendor/autoload.php';
{% endhighlight %}

At this point, assuming you have not yet created any test files, you should be able to run the phpunit suite with no 
results:

{% highlight bash %}
fraser@localhost database-testing $ vendor/bin/phpunit
PHPUnit 4.8.18 by Sebastian Bergmann and contributors.



Time: 94 ms, Memory: 4.50Mb

No tests executed!
{% endhighlight %}

# Creating The Test Classes

First, create a file `BookMapperTest.php` in the `tests/Persistence` folder.  The file should look something like this, 
with a test function for each of the functions that exist in the `BookMapper.php` file.

{% highlight php %}
<?php

namespace DatabaseTesting\Tests\Persistence\Mappers;

use DatabaseTesting\Tests\Persistence\DatabaseTestCase;

class BookMapperTest extends DatabaseTestCase
{
    /**
     * Prepare data set for database tests
     * 
     * @return \PHPUnit_Extensions_Database_DataSet_ArrayDataSet
     */
    public function getDataSet()
    {
        return $this->createArrayDataSet( array() );
    }    
    
    public function testFetchAll()
    {
        $this->markTestIncomplete( 'Not written yet.' );
    }

    public function testFetchByISBN()
    {
        $this->markTestIncomplete( 'Not written yet.' );
    }

    public function testInsert()
    {
        $this->markTestIncomplete( 'Not written yet.' );
    }

    public function testUpdate()
    {
        $this->markTestIncomplete( 'Not written yet.' );
    }

    public function testDelete()
    {
        $this->markTestIncomplete( 'Not written yet.' );
    }
}
{% endhighlight %}

Note two things about this file:  

1. the `getDataSet` function returns an empty dataset (this will be filled in later)
1. the class extends `DatabaseTestCase`, which you may have also noticed in the code structure image above
  
Create the `DatabaseTestCase.php` file next, in the `tests/Persistence` folder:

{% highlight php %}
<?php
namespace DatabaseTesting\Tests\Persistence;

use DatabaseTesting\Db\MysqlAdapter;

abstract class DatabaseTestCase extends \PHPUnit_Extensions_Database_TestCase
{
    // only instantiate pdo once for test clean-up/fixture load
    static private $pdo = null;

    // only instantiate PHPUnit_Extensions_Database_DB_IDatabaseConnection once per test
    private $conn = null;

    final public function getConnection()
    {
        if( $this->conn === null )
        {
            if( self::$pdo == null )
            {
                $dbAdapter = new MysqlAdapter( $GLOBALS[ 'DB_USER' ], $GLOBALS[ 'DB_PASSWD' ], $GLOBALS[ 'DB_DBNAME' ], $GLOBALS[ 'DB_HOST' ] );
                self::$pdo = $dbAdapter->getInstance();
            }
            $this->conn = $this->createDefaultDBConnection( self::$pdo );
        }

        return $this->conn;
    }
}
{% endhighlight %}

The `DatabaseTestCase` class extends the `PHPUnit_Extensions_Database_TestCase`, which is the base `phpunit/dbunit` test case class.  
The trait used by `PHPUnit_Extensions_Database_TestCase` contains the following abstract functions, requiring the test
class to implement:

 * `getConnection`
 * `getDataset`
 
By creating our own abstract version of the base test class (as recommended in the 
[PHPUnit Manual](https://phpunit.de/manual/4.8/en/database.html#database.tip-use-your-own-abstract-database-testcase)),
it allows you to define the database connection to be used in all of the tests.  In the example we are using the main 
database connection (as defined in `phpunit.xml.dist`), which is the same connection that the application itself uses.
This can also be adapted to use a specific `test` version of the database, depending on your application and it's 
environmental requirements.

Now if you run the tests, you should see 5 tests run, all incomplete:

{% highlight bash %}
fraser@localhost database-testing $ vendor/bin/phpunit
PHPUnit 4.8.18 by Sebastian Bergmann and contributors.

IIIII

Time: 107 ms, Memory: 5.25Mb

OK, but incomplete, skipped, or risky tests!
Tests: 5, Assertions: 0, Incomplete: 5.
{% endhighlight %}


# Creating Test DataSets

There are four documented ways to create datasets for database tests run with PHPUnit:  XML, YAML, CSV and Arrays.  The 
first three methods are natively supported, and the fourth can be supported with some custom code written.  Let's go over each.

**XML**

XML datasets can be used to store the test data used in the suite, with each XML tag inside the root node `<dataset>`
representing a row in a database table.  The tag name should be the name of the database table, and attributes should
be the column names in the table.

For example:

{% highlight xml %}
<?xml version="1.0" ?>
<dataset>
    <authors id="100" name="Mordecai Richler"/>
    <authors id="101" name="Farley Mowat"/>

    <books id="200" isbn="978-0671028473" title="The Apprenticeship of Duddy Kravitz" author_id="100" />
    <books id="201" isbn="978-0887769252" title="Jacob Two-Two Meets the Hooded Fang" author_id="100" />
    <books id="202" isbn="978-1550139891" title="The Farfarers" author_id="101" />
</dataset>
{% endhighlight %}

The XML file data is retrieved by a test class using the `createFlatXmlDataSet` function:

{% highlight php %}
<?php

namespace DatabaseTesting\Tests\Persistence\Mappers;

use DatabaseTesting\Tests\Persistence\DatabaseTestCase;

class BookMapperTest extends DatabaseTestCase
{
    /**
     * Prepare data set for database tests
     *
     * @return \PHPUnit_Extensions_Database_DataSet_AbstractDataSet
     */
    public function getDataSet()
    {
        return $this->createFlatXMLDataSet( __DIR__ . '/../../Fixtures/library-test.xml' );
    }
    
{% endhighlight %}

**YAML**

YAML datasets can also be used to store the test data used in the suite.  The structure of the YAML file is very
straightforward, with headings for each table and blocks below for each row in the table:

{% highlight yaml %}
authors:
  -
    id: 100
    name: "Mordecai Richler"
  -
    id: 101
    name: "Farley Mowat"
books:
  -
    id: 200
    isbn: "978-0671028473"
    title: "The Apprenticeship of Duddy Kravitz"
    author_id: 100
  -
    id: 201
    isbn: "978-0887769252"
    title: "Jacob Two-Two Meets the Hooded Fang"
    author_id: 100
  -
    id: 202
    isbn: "978-1550139891"
    title: "The Farfarers"
    author_id: 101
{% endhighlight %}

The YAML file data is retrieved by a test class using the `PHPUnit_Extensions_Database_DataSet_YamlDataSet` function:

{% highlight php %}
<?php

namespace DatabaseTesting\Tests\Persistence\Mappers;

use DatabaseTesting\Tests\Persistence\DatabaseTestCase;

class BookMapperTest extends DatabaseTestCase
{
    /**
     * Prepare data set for database tests
     *
     * @return \PHPUnit_Extensions_Database_DataSet_AbstractDataSet
     */
    public function getDataSet()
    {
        return new \PHPUnit_Extensions_Database_DataSet_YamlDataSet( __DIR__ . '/../../Fixtures/library-test.yml' );
    }
    
{% endhighlight %}

**CSV**

CSV datasets are slightly different in structure than XML and YAML datasets, as you must load each table individually 
with the data for each table contained in a separate file.  Maintaining the datasets this way, however, is much easier
if you are extracting the test data from an existing database, as the data dumps can be configured per file to match this
format.

`library-test-authors.csv`
{% highlight text %}
id,name
100,"Mordecai Richler"
101,"Farley Mowat"
{% endhighlight %}

`library-test-books.csv`
{% highlight text %}
id,isbn,title,author_id
200,"978-0671028473","The Apprenticeship of Duddy Kravitz",100
201,"978-0887769252","Jacob Two-Two Meets the Hooded Fang",100
202,"978-1550139891","The Farfarers",101
{% endhighlight %}

The CSV files are retrieved per table by a test class using the `PHPUnit_Extensions_Database_DataSet_CsvDataSet` function:

{% highlight php %}
<?php

namespace DatabaseTesting\Tests\Persistence\Mappers;

use DatabaseTesting\Tests\Persistence\DatabaseTestCase;

class BookMapperTest extends DatabaseTestCase
{
    /**
     * Prepare data set for database tests
     *
     * @return \PHPUnit_Extensions_Database_DataSet_AbstractDataSet
     */
    public function getDataSet()
    {
        $dataSet = new \PHPUnit_Extensions_Database_DataSet_CsvDataSet();
        $dataSet->addTable( 'authors', __DIR__ . '/../../Fixtures/library-test-authors.csv' );
        $dataSet->addTable( 'books', __DIR__ . '/../../Fixtures/library-test-books.csv' );
        
        return $dataSet;
    }
    
{% endhighlight %}

**Arrays**

Array datasets can be included directly in the `getDataSet` function, or can be loaded using included files similar to the 
methods used above.  However, you need to write your own data iterator which you can find a sample of in the 
[PHP Manual](https://phpunit.de/manual/4.8/en/database.html#database.array-dataset).  In the case of this example, the 
dataset is loaded using the custom `ArrayDataSet` class:

{% highlight php %}
<?php

namespace DatabaseTesting\Tests\Persistence\Mappers;

use DatabaseTesting\Tests\Persistence\ArrayDataSet;
use DatabaseTesting\Tests\Persistence\DatabaseTestCase;

class BookMapperTest extends DatabaseTestCase
{
    /**
     * Prepare data set for database tests
     *
     * @return \PHPUnit_Extensions_Database_DataSet_AbstractDataSet
     */
    public function getDataSet()
    {
        return new ArrayDataSet( array(
            'authors' => array(
                array( 'id' => 100, 'name' => 'Mordecai Richler' ),
                array( 'id' => 101, 'name' => 'Farley Mowat' )
            ),
            'books' => array(
                array( 'id' => 200, 'isbn' => '978-0671028473', 'title' => 'The Apprenticeship of Duddy Kravitz', 'author_id' => 100 ),
                array( 'id' => 201, 'isbn' => '978-0887769252', 'title' => 'Jacob Two-Two Meets the Hooded Fang', 'author_id' => 100 ),
                array( 'id' => 202, 'isbn' => '978-1550139891', 'title' => 'The Farfarers', 'author_id' => 101 )
            ),
        ) );
    }
    
{% endhighlight %}

# Writing The Tests

[fill in]