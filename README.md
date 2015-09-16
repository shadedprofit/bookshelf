# [bookshelf.js](http://bookshelfjs.org) [![Build Status](https://travis-ci.org/tgriesser/bookshelf.svg?branch=master)](https://travis-ci.org/tgriesser/bookshelf)

Bookshelf is a JavaScript ORM for Node.js, built on the [Knex](http://knexjs.org) SQL query builder. Featuring both promise based and traditional callback interfaces, it follows the Model & Collection patterns seen in [Backbone.js](http://backbonejs.com), providing transaction support, eager/nested-eager relation loading, polymorphic associations, and support for one-to-one, one-to-many, and many-to-many relations.

It is designed to work well with PostgreSQL, MySQL, and SQLite3.

The project is [hosted on GitHub](http://github.com/tgriesser/bookshelf/), and has a comprehensive [test suite](https://travis-ci.org/tgriesser/bookshelf).

## Introduction

Bookshelf aims to provide a simple library for common tasks when querying databases in JavaScript, and forming relations between these objects, taking a lot of ideas from the the [Data Mapper Pattern](http://en.wikipedia.org/wiki/Data_mapper_pattern). With a concise, literate codebase, Bookshelf is simple to read, understand, and extend. It doesn't force you to use any specific validation scheme, provides flexible and efficient relation/nested-relation loading, and first class transaction support. It's a lean Object Relational Mapper, allowing you to drop down to the raw knex interface whenever you need a custom query that doesn't quite fit with the stock conventions.

Bookshelf follows the excellent foundation provided by Backbone.js Models and Collections, using similar patterns, naming conventions, and philosophies to build a lightweight, easy to navigate ORM. If you know how to use Backbone, you probably already know how to use Bookshelf.

## Installation

You'll need to install a copy of [knex.js](http://knexjs.org/), and either mysql, pg, or sqlite3 from npm.

```js
$ npm install knex --save
$ npm install bookshelf --save

# Then add one of the following:
$ npm install pg
$ npm install mysql
$ npm install mariasql
$ npm install sqlite3
```

The Bookshelf library is initialized by passing an initialized [Knex](http://knexjs.org/) client instance. The [knex documentation](http://knexjs.org/) provides a number of examples for different databases.

```js
var knex = require('knex')({
  client: 'mysql',
  connection: {
    host     : '127.0.0.1',
    user     : 'your_database_user',
    password : 'your_database_password',
    database : 'myapp_test',
    charset  : 'utf8'
  }
});

var bookshelf = require('bookshelf')(knex);

var User = bookshelf.Model.extend({
  tableName: 'users'
});
```

This initialization should likely only ever happen once in your application. As it creates a connection pool for the current database, you should use the `bookshelf` instance returned throughout your library. You'll need to store this instance created by the initialize somewhere in the application so you can reference it. A common pattern to follow is to initialize the client in a module so you can easily reference it later:

```js
// In a file named something like bookshelf.js
var knex = require('knex')(dbConfig);
module.exports = require('bookshelf')(knex);

// elsewhere, to use the bookshelf client:
var bookshelf = require('./bookshelf');

var Post = bookshelf.Model.extend({
  // ...
});
```

## Examples

Here is an example to get you started:

```js
var knex = require('knex')({client: 'mysql', connection: process.env.MYSQL_DATABASE_CONNECTION });
var bookshelf = require('bookshelf')(knex);

var User = bookshelf.Model.extend({
  tableName: 'users',
  messages: function() {
    return this.hasMany(Posts);
  }
});

var Posts = bookshelf.Model.extend({
  tableName: 'messages',
  tags: function() {
    return this.belongsToMany(Tag);
  }
});

var Tag = bookshelf.Model.extend({
  tableName: 'tags'
})

User.where('id', 1).fetch({withRelated: ['posts.tags']}).then(function(user) {

  console.log(user.related('posts').toJSON());

}).catch(function(err) {

  console.error(err);

});
```

## Plugins

 - [Registry](https://github.com/tgriesser/bookshelf/wiki/Plugin:-Model-Registry): Register models in a central location so that you can refer to them using a string in relations instead of having to require it every time. Helps deal with the challenges of circular module dependencies in Node.
 - [Virtuals](https://github.com/tgriesser/bookshelf/wiki/Plugin:-Virtuals): Define virtual properties on your model to compute new values.
 - [Visibility](https://github.com/tgriesser/bookshelf/wiki/Plugin:-Visibility): Specify a whitelist/blacklist of model attributes when serialized toJSON.

## Support

Have questions about the library? Come join us in the [#bookshelf freenode IRC channel](http://webchat.freenode.net/?channels=bookshelf) for support on [knex.js](http://knexjs.org/) and bookshelf.js, or post an issue on [Stack Overflow](http://stackoverflow.com/questions/tagged/bookshelf.js) or in the GitHub [issue tracker](https://github.com/tgriesser/bookshelf/issues).

## F.A.Q.

### Can I use standard node.js style callbacks?

Yes - you can call `.asCallback(function(err, resp) {` on any "sync" method and use the standard `(err, result)` style callback interface if you prefer.

### My relations don't seem to be loading, what's up?

Make sure you check that the type is correct for the initial parameters passed to the initial model being fetched. For example `new Model({id: '1'}).load([relations...])` will not return the same as `Model({id: 1}).load([relations...])` - notice that the id is a string in one case and a number in the other. This can be a common mistake if retrieving the id from a url parameter.

This is only an issue if you're eager loading data with load without first fetching the original model. `Model({id: '1'}).fetch({withRelated: [relations...]})` should work just fine.

### My process won't exit after my script is finished, why?

The issue here is that Knex, the database abstraction layer used by Bookshelf, uses connection pooling and thus keeps the database connection open. If you want your process to exit after your script has finished, you will have to call `.destroy(cb)` on the `knex` property of your `Bookshelf` instance or on the `Knex` instance passed during initialization. More information about connection pooling can be found over at the [Knex docs](http://knexjs.org/#Installation-pooling).

### How do I debug?

If you pass `{debug: true}` as one of the options in your initialize settings, you can see all of the query calls being made. Sometimes you need to dive a bit further into the various calls and see what all is going on behind the scenes. I'd recommend [node-inspector](https://github.com/dannycoates/node-inspector), which allows you to debug code with `debugger` statements like you would in the browser.

Bookshelf uses its own copy of the "bluebird" promise library, you can read up here for more on debugging these promises... but in short, adding:

    process.stderr.on('data', function(data) {
      console.log(data);
    });

At the start of your application code will catch any errors not otherwise caught in the normal promise chain handlers, which is very helpful in debugging.

### How do I run the test suite?

The test suite looks for an environment variable called `BOOKSHELF_TEST` for the path to the database configuration. If you run the following command: `$ export BOOKSHELF_TEST='/path/to/your/bookshelf_config.js'`, replacing with the path to your config file, and the config file is valid, the test suite should run with npm test.

Also note that you will have to create the appropriate database(s) for the test suite to run. For example, with MySQL, you'll need to run the command `create database bookshelf_test;` in addition to exporting the correct test settings prior to running the test suite.

### Can I use Bookshelf outside of Node.js?

While it primarily targets Node.js, all dependencies are browser compatible, and it could be adapted to work with other javascript environments supporting a sqlite3 database, by providing a custom [Knex adapter](http://knexjs.org/#Adapters).
