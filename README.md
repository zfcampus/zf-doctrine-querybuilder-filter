Soliant Consulting 
==================

Apigility for Doctrine
----------------------

This library has three parts.  

1. An Admin tool to create an Apigility-enabled module with support for all 
Doctrine Entities in scope.

2. An API server AbstractService class to handle API interactions from resources
created with the Admin tool.

3. An API client [Object Relational Mapper]
(https://en.wikipedia.org/wiki/Object-relational_mapping) based on the 
[Doctrine Common](http://www.doctrine-project.org/projects/common.html) 
project library which works in tandem with a Doctrine ORM class to make interacting
with your entities seemless across the API.  See the README.CLIENT.md


Installation
------------
  1. edit `composer.json` file with following contents:

     ```json
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/TomHAnderson/soliantconsulting-apigility"
        }
    ],
    "require": {
        "soliantconsulting/soliantconsulting-apigility": "dev-master"
    }
     ```
  2. install composer via `curl -s http://getcomposer.org/installer | php` (on windows, download
     http://getcomposer.org/installer and execute it with PHP)
  3. run `php composer.phar install`


Doctrine Entity Configuration
-----------------------

Doctrine entities are hydrated with the DoctrineEntity hydrator so no special configuration of your entities is needed to use this tool.


Creating the Apigility-enabled module
-------------------------------------

The Admin tool can create an Apigility-enabled module with the Doctrine entities in scope.
To enable the Admin include ```'SoliantConsulting\Apigility',``` in your 
development.config.php configuration.

All entities managed by the object manager will be available to build into a resource.  

Browse to ```/soliant-consulting/apigility/admin``` to begin.  On this page you will enter 
the name of a new module which does not already exist.  When the form is submitted
the module will be created.

The next page allows you to select entities from the object manager to build into 
resources.  You may change your object manager and refresh entities for that object
manager.  Check the entities you want then submit the form.

Done.  Your new module is enabled in your application and you can start making API 
requests.  

The route for an entity named DataModule\Entity\UserData is
```/api/user_data``` or, if using namespaced routes, ```/api/data_module/entity/user_data``` 
After going through the above process your API should be working.


Handling Embedded Resources
---------------------------

If you choose you my supress embedded resources by setting
$config[zf-hal][render_embedded_resources] to false.  Doing so
returns only links to embedded resources instead of their full details.
This setting is useful for avoiding circular references.


Collections 
===========

The API created with this library implements full featured and paginated 
collection resources.  A collection is returned from a GET call to a entity endpoint without
specifying the id.  e.g. ```GET /api/data_module/entity/user_data```

Reserved GET variables

```
    orderBy
    query
```

Sorting Collections
-------------------

Sort by columnOne ascending

```
    /api/user_data?orderBy%5BcolumnOne%5D=ASC
```

Sort by columnOne ascending then columnTwo decending

```
    /api/user_data?orderBy%5BcolumnOne%5D=ASC&orderBy%5BcolumnTwo%5D=DESC
```


Querying Collections
--------------------

Queries are not simple key=value pairs.  The query parameter is a key-less array of query 
definitions.  Each query definition is an array and the array values vary for each query type.

Each query type requires at a minimum a 'type' and a 'field'.  Each query may also specify
a 'where' which can be either 'and' or 'or'.  Embedded logic such as and(x or y) is not supported.

Building HTTP GET query with PHP.  Use this to help build your queries.

PHP Example
```php
    echo http_build_query(
        array(
            'query' => array(
                array('field' =>'cycle', 'where' => 'and', 'type'=>'between', 'from' => 1, 'to'=>100),
                array('field'=>'cycle', 'where' => 'and', 'type' => 'decimation', 'value' => 10)
            ),
            'orderBy' => array('columnOne' => 'ASC', 'columnTwo' => 'DESC')
        )
    );
```

Javascript Example
```js
$(function() {
    $.ajax({
        url: "http://localhost:8081/api/db/entity/user_data",
        type: "GET",
        data: {
            'query': [
            {
                'field': 'cycle',
                'where': 'and',
                'type': 'between',
                'from': '1',
                'to': '100'
            },
            {
                'field': 'cycle',
                'where': 'and',
                'type': 'decimation',
                'value': '10'
            }
        ]
        },
        dataType: "json"
    });
});
```
    

Available Query Types
---------------------

Equals

```php
    array('type' => 'eq', 'field' => 'fieldName', 'value' => 'matchValue')
```

Not Equals

```php
    array('type' => 'neq', 'field' => 'fieldName', 'value' => 'matchValue')
```

Less Than

```php
    array('type' => 'lt', 'field' => 'fieldName', 'value' => 'matchValue')
```

Less Than or Equals

```php
    array('type' => 'lte', 'field' => 'fieldName', 'value' => 'matchValue')
```

Greater Than

```php
    array('type' => 'gt', 'field' => 'fieldName', 'value' => 'matchValue')
```

Greater Than or Equals

```php
    array('type' => 'gte', 'field' => 'fieldName', 'value' => 'matchValue')
```

Is Null

```php
    array('type' => 'isnull', 'field' => 'fieldName')
```

Is Not Null

```php
    array('type' => 'isnotnull', 'field' => 'fieldName')
```

In

```php
    array('type' => 'in', 'field' => 'fieldName', 'values' => array(1, 2, 3))
```

NotIn

```php
    array('type' => 'notin', 'field' => 'fieldName', 'values' => array(1, 2, 3))
```

Like (% is used as a wildcard)

```php
    array('type' => 'like', 'field' => 'fieldName', 'value' => 'like%search')
```

Not Like (% is used as a wildcard)

```php
    array('type' => 'notlike', 'field' => 'fieldName', 'value' => 'notlike%search')
```

Between

```php
    array('type' => 'between', 'field' => 'fieldName', 'from' => 'startValue', 'to' => 'endValue')
````

Decimation (mod(field, value) = 0 e.g. value = 10 fetch one of every ten rows)

```php
    array('type' => 'decimation', 'field' => 'fieldName', 'value' => 'decimationModValue')
```
