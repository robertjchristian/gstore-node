# Datastools (work in progress)
Datastools is a Google Datastore entities modeling library for Node.js inspired by Mongoose and built on top of the **gcloud-node** library.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Motivation](#motivation)
  - [Installation](#installation)
  - [Getting started](#getting-started)
- [Schema](#schema)
  - [Creation](#creation)
  - [Properties types](#properties-types)
  - [Values validations](#values-validations)
  - [Other Schema options](#other-schema-options)
    - [optional](#optional)
    - [default](#default)
    - [excludedFromIndex](#excludedfromindex)
- [Model](#model)
  - [Creation](#creation-1)
  - [Methods](#methods)
    - [Save](#save)
  - [Queries](#queries)
    - [gcloud queries](#gcloud-queries)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Motivation
The Google Datastore is an amazing fast, reliable and flexible database for today's modern apps. But it's flexibility and *schemaless* nature can 
sometimes lead to a lot of duplicate code to **validate** the properties to save. The **pre & post 'hooks'** found in Mongoose are also of great value when it comes to work with entities on a NoSQL database.

Datastools enhances the experience to work with entities of Googe Datastore.
It is still in in active development (**no release yet**).

### Installation
 ```
 npm install datastools --save
 ```
 
### Getting started
(For info on how to configure gcloud go here: https://googlecloudplatform.github.io/gcloud-node/#/docs/v0.34.0/gcloud?method=gcloud)
 ```
 var configGcloud = {...your config here};
 var gcloud       = require('gcloud')(configGcloud);
 var ds           = gcloud.datastore(configDatastore);
 
 var datastools = require('datastools');
 datastools.connect(ds);
 ```

## Schema
### Creation
```
var datastools = require('datastools');
var Schema     = datastools.Schema;

var entitySchema = new Schema({
    name:{},
    lastname:{},
    ...
});
```

### Properties types
Valid property types are
- 'string'
- 'number'
- 'boolean'
- 'datetime' (valid format: 'YYYY-MM-DD' | 'YYYY-MM-DD 00:00:00' | 'YYYY-MM-DD 00:00:00.000' | 'YYYY-MM-DDT00:00:00')
- 'array'

```
var entitySchema = new Schema({
    name     : {type: 'string'},
    lastname : {},  // if nothing is passed, no type validation occurs (anything goes in!)
    age      : {type: 'number'},
    hasPaid  : {type: 'boolean'},
    createdOn: {type: 'datetime'},
    tags     : {type: 'array'}
});
```

### Properties values validations
Datastools uses the great validator library (https://github.com/chriso/validator.js) to validate input values so you can use any of the validations from that library.

```
var entitySchema = new Schema({
    email  : {validate: 'isEmail'},
    website: {validate: 'isURL'},
    color  : {validate: 'isHexColor'},
    ...
});
```
### Other properties options
#### optional
By default if a property value is not defined it will be set to null or its default value (see below) if any. If you don't want this behaviour you can set it as *optional* and if now value is passed nothing will be saved on the entity.

#### default
You can set a default value for the property is no value has been passed.

#### excludedFromIndex
By default all properties are **included** in the Datastore indexes. If you don't want some properties to be indexed set their 'excludedFromIndex' property to false.

```
// Properties options example
var entitySchema = new Schema({
    name    : {type: 'string'},
    lastname: {excludedFromIndex: true},
    website : {validate: 'isURL', optional: true},
    modified: {type: 'boolean', default: false},
    ...
});
```

### Schema options
#### validateBeforeSave (default true)
To disable any validation before save/update, set it to false

#### entities
**simplifyResult** (default true).
By default the results coming back from the Datastore are serialized into a more readable object format. If you want the full response that includes both the Datastore Key & Data, set simplifyResult to false. This option can be set on a per query basis (see below).

```
// Schema options example
var entitySchema = new Schema({
    name : {type: 'string'}
}, {
    validateBeforeSave : false,
    entities : {
        simplifyResult : false
    }
});
```

## Model
### Creation

```
var datastools = require('datastools');
var Schema     = datastools.Schema;

var entitySchema = new Schema({
    name:{},
    lastname:{},
    email:{}
});

var model = datastools.model('EntityName', entitySchema);
```

### Methods
#### Save

```
var datastools = require('datastools');

var blogPostSchema = new datastools.Schema({
    title : {type:'string'},
    createdOn : {type:'datetime'}
});

var BlogPost = datastools.model('BlogPost', blogPostSchema);

var data = {
    title : 'My first blog post',
    createdOn : new Date()
};
var blogPost = new BlogPost(data);

blogPost.save(function(err) {
    if (err) {
        // deal with err
    }
    console.log('Great! post saved');
});
```

### Queries
#### gcloud queries
Datastools is built on top of [gcloud-node](https://googlecloudplatform.github.io/gcloud-node/#/docs/v0.34.0/datastore/query) so you can execute any query from that library.

````
var BlogPost = datastools.model('BlogPost', schema);

var query = BlogPost.query();

query.filter('name', '=', 'John')
     .filter('age', '>=', 4)
     .order('lastname', {
         descending: true
     });

query.run(function(err, entities, info) {
    if (err) {
        // deal with err
    }
    console.log('Entities found:', entities);
});
```

#### list
Shortcut for listing the entities. For complete control (pagination, start, end...) use the above gcloud queries. List queries are meant to quickly list entites with predefined settings.
Currently it support the following settings:
- limit
- order
- select
- ancestors
- filters (default operator is "=" and does not need to be passed

```
var blogPostSchema = new datastools.Schema({
    title : {type:'string'},
    createdOn : {type:'datetime'}
});

var querySettings = {
    limit    : 10,
    order    : {property: 'title'},
    select   : 'title'
    ancestors: ['Parent', 123],  // will add a hasAncestor filter
    filters  : ['title', 'My first post'] // operator defaults to "="
};

blogPostSchema.queries('list', querySettings);
var BlogPost = datastools.model('BlogPost', blogPostSchema);

...

// anywhere in your Controllers
BlogPost.list(function(err, entities) {
    if (err) {
        // deal with err
    }
    console.log(entities);
});
```

Order, Select & filters settings can also be **arrays**

```
var querySettings = {
    orders  : [{property: 'title'}, {property:'createdOn', descending:true}]
    select  : ['title', 'createdOn'],
    filters : [['title', 'My first post'], ['createdOn', '<',  new Date()]]
};
```

The settings can be overriden anytime by passing another settings object as first parameter

```
var newSettings = {
    limit : 20
};

BlogPost.list(newSettings, function(err, entities) {
    if (err) {
        // deal with err
    }
    console.log(entities);
});
```
