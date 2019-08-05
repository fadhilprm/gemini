<p align="center">
   <img src="./gemini_logo.png" height="150">
</p>

___

![License](https://img.shields.io/github/license/h4t0n/gemini.svg)
[![Build Status](https://travis-ci.org/h4t0n/gemini.svg?branch=master)](https://travis-ci.org/h4t0n/gemini)
[![Twitter](https://img.shields.io/badge/Twitter-@h4t0n-blue.svg?style=flat)](http://twitter.com/h4t0n)
[![Gitter](https://img.shields.io/gitter/room/gemini-framework/general)](https://gitter.im/gemini-framework/general)
![Last Commit](https://img.shields.io/github/last-commit/h4t0n/gemini.svg)

Gemini make it easy to create full backends for modern web/mobile application, microservice and MVP.

Gemini is a backend REST framework to automatically create CRUD REST APIs from scratch starting from a simple Schema
 definition called Gemini DSL. Briefly Gemini automatically handles for you:
* **Persistence**: creating all the storage and entity managers
* **API controllers**: creating all the common REST methods 
* **Swagger API documentation**: creating the openapi language-agnostic interface to RESTful APIs 
* **Authentication**: by using Spring OAuth2 and JWT tokens
* **API callbacks**: to add business logic with ease


**NB:** Gemini is not a code generator, all the resources are handled dynamically. 

<p align="center">
   <img src="./gemini_hiw.gif" height="400">
</p>

## Index

* [Features](#features)
* [Quick Start & Setup](#quick-start--setup)
    * [Environment Setup](#environment-setup)
    * [Start Gemini](#start-gemini)
    * [Once Started](#once-started)
* [DSL and APIs](#dsl-and-apis)
    * [Entity and Logical Keys](#entity-and-logical-keys)
    * [Primitive Types](#primitive-types)
    * [Date And Time](#date-and-time)
    * [Entity Reference](#entity-reference-types)
* [Modules](#modules)
    * [Core Module](#gemini-core)
        * [Entity and Field](#entity-and-field)
        * [Gemini Meta Data](#gemini-meta-data)
            * [UUID-V3](#uuid-v3)
    * [Auth Module](#auth-module)
    * [Runtime Module](#runtime-module)
* [Contribute](#contribute)
* [License](#license)

## Features
* **REST Resources** (Entities) are defined with the simple **Gemini DSL**
    * Support for simple data types (TEXT, NUMBERS, DATES, TIME ...)
    * Support for custom and complex types (FILE, IMAGE, ...) // WIP work in progress
* **Automagical generation** of REST CRUD APIs for each Resource
    * POST - PUT - GET - DELETE are available out of the box
    * Support for custom routes (Gemini Services available)  
* No need to handle Entities and Relations persistence
    * Gemini Entity Manager check records and relations (by using Entity Logical Keys)
    * Gemini Persistence Manager automatically handle storage and data
        * Postgresql Driver available
* REST openApi documentation 


For example lets use the Gemini DSL to define....

```text
# ..two simple Entities: Book and Author. Where * is used to specify the logical key

ENTITY Book {
      TEXT        isbn    *
      TEXT        name
      AUTHOR      author
      DECIMAL     price
}
  
ENTITY Author {
      TEXT    name        *
      DATE    birthdate
}

# The following routes are automatically available  
# /api/book             GET (list) / POST (single resouce)
# /api/book/{isbn}      GET / PUT / DELETE (single resource)

# /api/author           GET (list) / POST (single resouce)
# /api/author/{isbn}    GET / PUT / DELETE (single resource)

# Where Book has a reference to Author by using its name (logical key)
```

**ATTENTION:** Gemini is a WIP and it is not ready for production environments.

## Quick start & Setup
Gemini has a modular structure but the artifact is always a Spring Boot application that you can run everywhere.

### Environment Setup 
#### Requirements
You can setup Gemini in your local machine installing the following requirements:
* Postgresql 11+
* Java 9+
* Gradle 5+

But the easiest way is to use docker and avoid to install other software except Java.

#### Docker Development Environment
With docker you can setup a full development environment with. Inside the ``docker/dev/`` directory you have the
docker-compose configuration and a real working directory environment that you can use or modify. The default environment
has the following services:
* **Postgresql 11+** database 
    * port=5432 - dbname=gemini - user=gemini - pwd=gemini
    * Gemini ``wd/application.properties`` already uses previous parameters
* **PgAdmin** to have a full control of postgres server
    * Exposed on port 8081 (http://127.0.0.1:8081)
    * Login Username and Password are both ``gemini``
    * The ``Gemini - Docker`` server configuration is provided out of the box just type the ``gemini`` password the first time
* **Swagger UI** to consume the Gemini APIs
    * Available on port 8082 (http://127.0.0.1:8082)
    * Gemini autogenerated openapi definitions are automatically loaded where:
        * ALL definition lists all the Gemini resources (also metadata and core resources)
        * RUNTIME definiton lists only the resurces defined in the ``RUNTIME.at`` DSL file

Start docker services is simple, docker compose works for you for all the volumes and variables mapping
```bash
# from gemini root
$ cd docker/dev
$ docker-compose up -d
```

You can find a video tutorial [here](https://vimeo.com/352005400).

### Start Gemini

When all services are up (only postgresql is required, but use docker is suggested to consume API with swagger) you can
run Gemini as a normal Spring Boot application. You can build the standalone executable or use your preferred IDE. The 
only things to remember is to specify the ``docker/dev/wd/`` as the Java working directory in order to use the already
crafted ``wd/application.properties``.

##### From IDE
First of all open/load the Gemini root gradle configuration. Gemini is made of modules that may have different main
or entry points. ``gemini-postgresql`` for example provides two main:
* *it.at7.gemini.boot.PostgresqlGeminiMain* to start Gemini with Core and Auth Module

You can disable authentication with ``gemini.auth = false`` in ``application.properties``.

**NB:** you can also develop you modules and you Gemini only as a dependency. Contact me if you need it now. I will made
a dedicated documentation section to custom module development (it is a big part of Gemini). Custom modules are required
to provide specific business logic to your APIs (it is a work in progress).

##### Build Executable
Gemini is built on SpringBoot that allow to generate a standalone executable file with all dependencies.
```bash
# from gemini root - to build a standalone jar
gradle bootJar

# from gemini root - to build a standalone EXECUTABLE jar
gradle executableJar
cd gemini-postgresql/dist
```
 
Now you can run the jar. 
```
# for standalone (bootJar) you need to use java -jar 
./gemini-postgresql-0.2.x-standalone.jar

# while for executable (executableJar) you can run it
# works on linux based machines - only some platform supported - take a look at Spring documentation
./gemini-postgresql-0.2.x-executable.jar
```

### Once Started
Gemini automatically setup its internal structures and creates the ``schema/RUNTIME.at`` abstract type file in the
working directory. Now you can customize this file with all your entities, by using the Gemini DSL.
Then restart the application to see your APIs in action.

If you want to easily navigate APIs you can use the Swagger openapi tools. Gemini automatically generate the openapi 3
json file `openapi/schema/runtime.json`

If you are using Docker all the services are already configured to see all the generated files and specifications, enjoy.

## DSL and APIs

### Entity and Logical Keys

Entities are the most important element of Gemini. They defines:
* REST Resources routes (via the EntityName)
    * `/api/{entityName}` is the default route
    * `GET api/{entityName}` for the list of available resources
    * `POST api/{entityName}` to insert a new one
* Body Fields (and their type)
    * JSON is the out of the BOX supported content type
* Entity Resource Record logical key (with `*`)
    * `/api/{entityName}/{resourseLogicalKey}` is the target route for a single resource
    * `GET /api/{entityName}/{resourseLogicalKey}` to get the single resource
    * `PUT /api/{entityName}/{resourseLogicalKey}` to update/modify the single resource

```
# For example this is a valid Entity

ENTITY SimpleText {
    TEXT   code *
}

# lets insert a resource
$ curl -d '{"code":"thelk"}' -H "Content-Type: application/json" -X POST http://127.0.0.1:8090/api/simpletext -i

  HTTP/1.1 200
  Content-Type: application/json;charset=UTF-8
  
  {"code":"thelk"}
 
# now we can query the previous inserted one
$ curl http://127.0.0.1:8090/api/simpletext/thelk -i
  
  HTTP/1.1 200
  Content-Type: application/json;charset=UTF-8

  {"code":"thelk"}
```

### Primitive Types

Primitive types are numbers, strings and booleans and their aliases.

```
ENTITY PrimitiveTypes {
    TEXT            code *          // TEXT is the single type for Strings  
    DOUBLE          double
    DECIMAL         anotherDouble
    NUMBER          anyNumber       // (any number, no matter if it is a floating point or not)
    LONG            long
    QUANTITY        anotherLong
    BOOL            isOk
}
```

#### Numbers
In Json notation *Number* is a single type. In Gemini it is the same, but it adds some semantic to each field (useful
for validation)

* **DOUBLE** or **DECIMAL**: floating point numbers
* **LONG** or **QUANTITY**: integer numbers
* **NUMBER**: any number no matter if it has decimals or not 

*they may be extended (for example adding only naturals numbers and so on)*

#### Strings and Text
Any string field can be defined with the **TEXT** type. No matter its size, it is a JSON string.

#### Boolean
Boolean is *true* or *false* and can be defined with the **BOOL** type

#### API example

```
$ curl -H "Content-Type: application/json" -X POST http://127.0.0.1:8090/api/primitivetypes -i
  -d '{"code":"logicalKey","anotherDouble":7.77,"double":7.77,"isOk":true,"anotherLong":7,"anyNumber":70,"long":7}'
  
   HTTP/1.1 200
   Content-Type: application/json;charset=UTF-8
   
   {"code":"logicalKey","anotherDouble":7.77,"double":7.77,"isOk":true,"anotherLong":7,"anyNumber":70,"long":7}
```


### Date And Time
Dates and Times are Strings (in JSON notation). Gemini uses the ISO 8601 Data elements and interchange formats standard. 

```
ENTITY DatesEntity {
    TEXT        code *
    DATE        aDate
    TIME        aTime
    DATETIME    aDateTime
}
```

#### Date
**DATE** is the type you can use for a simple local date (no need of timezone).
Out of the box supported JSON API format is `yyyy-MM-dd`.

#### Time
**TIME** is the type you can use for ISO-8601 standard time. ISO-TIME is the out of the box supported format.
So for example the JSON String `09:41:43.973Z` is a valid time.

#### Datetime
**DATETIME** can be used for a date time field. It is a full ISO-8601 standard ISO-DATE-TIME.
So for example `2019-05-19T09:41:30.000Z` is a valid value.

#### API example

```
$ curl -H "Content-Type: application/json" -X POST http://127.0.0.1:8090/api/datestypes -i -d '{"aTime":"11:11:05.759Z","code":"lkdate","aDate":"2019-05-19","aDateTime":"2019-05-19T11:09:58Z"}'
    
    HTTP/1.1 200
    Content-Type: application/json;charset=UTF-8

    {"aTime":"11:11:05.759Z","code":"lkdate","aDate":"2019-05-19","aDateTime":"2019-05-19T11:09:58Z"}
```

### Entity Reference Types

Each defined Entity (with a logical key) is itself a Gemini Type. 

```
ENTITY Tag {
    TEXT    name    *
    TEXT    description
}

# Tag is a type that we can use in the Post Entity

ENTITY Post {
    LONG postId *
    TEXT message
    Tag  tag
}
```

#### API Example - Insert Tag and Post

First of all we need at least one tag.
```
$ curl -H "Content-Type: application/json" -X POST http://127.0.0.1:8090/api/tag -i -d '{"name":"italy","description":"We ❤️ Italy"}'

    HTTP/1.1 200
    Content-Type: application/json;charset=UTF-8

    {"name":"italy","description":"We ❤️ Italy"}
```

To insert the POST only the logical key of Tag is required. 

```
$ curl -H "Content-Type: application/json" -X POST http://127.0.0.1:8090/api/post -i -d '{"postid":1,"message":"Rome", "tag": "italy"}'

    HTTP/1.1 200
    Content-Type: application/json;charset=UTF-8

    {"tag":"italy","postId":1,"message":"Rome"}
```

#### API Example - Post with unknown Tag

What if we try to insert a Post with an unknown tag?. Not found error!!!

```
$ curl -H "Content-Type: application/json" -X POST http://127.0.0.1:8090/api/post -i -d '{"postid":2,"message":"Rome", "tag": "unknown"}'

    HTTP/1.1 404
```

### Arrays

Arrays are a Work in Progress. Gemini will supports array out of the box either from primitive types or complex types. 
Actually is supported only the TEXT array.

```
ENTITY Messages {
    LONG messageId *
    [TEXT] message
}
```

## Modules

### Gemini CORE Module
Core is the main module of Gemini and it defines some features, and predefined entities (and fields) available also
for developers. Take a look at the `Core.at` abstract type schema.

#### ENTITY and FIELD


Gemini tracks all the Entities using the ENTITY Entity. All fields instead are stored in the FIELD Entity. 
It looks like a work game, but let's query them.

```
# The list of all available entities
$ curl http://127.0.0.1:8090/api/entity -i

# The list of all available fields
$ curl http://127.0.0.1:8090/api/field -i
```

By convention, entity names are always expressed (and stored) in UPPERCASE notation. While fields are handled in
LOWERCASE notation. 

#### Gemini META data
Gemini handle meta data for each operation performed on a single EntityRecord. You can retrieve these meta information
using the Gemini header.

```
Gemini: gemini.api

The JSON response has the following schema
{
    "meta": {...}
    "data": {...}
}
```

NB: Core metadata are defined inside the `Core.at` schema and by default information about created and modified time
are automatically stored by the framework.


Let's query for example the `PrimitiveTypes` defines earlier.

```
$ curl -H "Gemini: gemini.api" http://127.0.0.1:8090/api/primitivetypes/test -i
    HTTP/1.1 200
    Gemini: gemini.api
    Content-Type: application/json;charset=UTF-8

    {
        "data":{"code":"test","anotherDouble":0.01,"double":0.2,"isOk":true,"anotherLong":2,"anyNumber":0.01,"long":1},
        "meta":{
            "created":"2019-05-19T14:49:57.765151Z",
            "modified":"2019-05-19T14:49:57.765151Z",
            "uuid":"97907b5d-854a-34ff-92ef-251055adba2a"
         }
     }
```

##### UUID-V3

Gemini automatically create a unique ID for each record by using UUID-v3 ids generated starting from the logical key.

You can query resources also by their UUID, especially when the logical key is made by multiple fields.

```
# note that there are not meta because we have not inserted the Gemini header in the request
# The uuid the same returned from the previuous example

$ curl  http://127.0.0.1:8090/api/primitivetypes/97907b5d-854a-34ff-92ef-251055adba2a -i

    HTTP/1.1 200
    Content-Type: application/json;charset=UTF-8

    {"code":"test","anotherDouble":0.01,"double":0.2,"isOk":true,"anotherLong":2,"anyNumber":0.01,"long":1}

```

Take a look at this article for some clarification [Rest API: UUID-V3 is the right way](https://medium.com/@h4t0n/rest-api-uuid-v3-is-the-right-way-3ca0695610dc)

### Auth Module
// TODO - Soon in v0.2.x

### Runtime Module
// TODO

## Contribute
Gemini is in early stage of development. The best way to contribute is to checkout the project and try it. Then we can
speak about new features, improvements and roadmap on gitter or social platforms.

## License
Apache License 2.0
