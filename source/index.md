---
title: API Reference

language_tabs:
  - javascript

toc_footers:
  - <a href='https://neti.am'>Project Website</a>
  - <a href='https://github.com/netiam/netiam'>Github Repository</a>

includes:
  - es2015

search: true
---

# Introduction


[![Build Status](https://travis-ci.org/netiam/netiam.svg)](https://travis-ci.org/netiam/netiam)
[![Dependencies](https://david-dm.org/netiam/netiam.svg)](https://david-dm.org/netiam/netiam)
[![Code Climate](https://codeclimate.com/github/netiam/netiam/badges/gpa.svg)](https://codeclimate.com/github/netiam/netiam)
[![Dependencies](https://david-dm.org/netiam/netiam.svg)](https://david-dm.org/netiam/netiam)

This REST API library addresses some issues I had with API designs over the
last years. It does not claim to provide a full featured solution and to be
honest, it might never will. Nevertheless, someone might find this library
useful. It works as *connect* [middleware](https://github.com/senchalabs/connect)
and you should be able to use it with any compatible infrastructure
(e.g. [express](http://expressjs.com/)).

# Get it

Use `npm` to install `netiam` into your projects `package.json`.

`npm i -S netiam`

> npm install

```bash
npm i -S netiam
```

# Routes

```js
export default function(app) {
  let router = express.Router()
  
  users(router)

  app.use('/v1', router)
}
```

In order to follow [RESTful best practices](https://github.com/tfredrich/RestApiTutorial.com/raw/master/media/RESTful%20Best%20Practices-v1_2.pdf),
it is recommended that you prefix your endpoints with a version part.

# Middleware

```js
app.get(
  '/users',
  api()
    .auth({collection: User})
    .rest({collection: User})
    .acl.res({
      collection: User,
      asserts: asserts.owner('_id')
    })
    .map.res({_id: 'id'})
    .jsonapi({collection: User})
)
```

The `netiam` API middleware is working just as any other `connect` middleware
you have might used in the past. The example shows you, how it can be used
together with `express`.

# Plugins

## Anatomy of a plugin

There are different types of plugins but they all share the same anatomy.
A plugin has a `construction` part and a `dispatch` function.

> Anatomy of a plugin

```js
function myPlugin(spec) {
  // …do your setup stuff here
  
  return function (req, res) {
    // dispatch
  }
}
```

## Register custom plugin

If you want to create your own plugin, you have to register it, so that the
dispatcher can put it on an endpoints stack.

> Register custom plugin

```js
import plugins from 'netiam/plugins'
import myplugin from './myplugin'

plugins.register(myplugin, 'myplugin')
```

Be aware of the fact, that you must register the plugin before you can use it
on any endpoint (e.g. main.js).

## Dual purpose plugin

There is one more category of plugins, those who work on the `request` and on
the `response` part of a full HTTP request/response loop. The `acl` plugin is
of this type. Basically, this is just for convenience. Someone might create two
different plugins, one that filters input and a second one which filters the
output. But as those tasks are similar and commonly used together, you might
want to pack it into a file.

> Export from a req/res plugin

```js
// Return an object with 2 methods
export default Object.freeze({
  req: request,
  res: response
})
```

## ACL

<aside class="warning">
You should never use the <code>acl</code> plugin without the <code>auth</code> plugin. All requests
will be treated as the user is of role <code>GUEST</code>. This might result in unexpected
behavior.
</aside>

> The `acl` middleware

```js
app.get(
  '/users',
  api()
    .auth({collection: User})
    .rest({collection: User})
    .acl.res({
      settings: require('./users.acl')
    })
    .jsonapi({collection: User})
)
```

The ACL plugin is a `dual purpose` plugin. It allows you to filter the input and
output of a resource endpoint. You can use this to set fine-grained permissions
for your users and resources.

### ACL Settings

In order to get this plugin working, you must provide a `settings` option which
must contain a valid ACL object.

> ACL settings on resource and fields level

```js
export default {
  'resource': {
    'ALLOW': {
      'ADMIN': 'CRUD',
      'USER': 'R',
      'GUEST': 'R'
    }
  },
  'fields': {
    '*': {
      'ALLOW': {
        'ADMIN': 'CRUD'
      }
    },
    'password': {
      'ALLOW': {
        'OWNER': 'U'
      }
    }
  }
}
```

### Nesting

ACL settings do support nesting of resources. If you expand a path of a resource
with the `expand` query parameter (`?expand=path`), you should filter the
nested path too. You can simply reuse existing ACL settings by referencing to
them. The following example shows how you can use ACL on a user resource with
a nested `projects` path.

> Referencing projects ACL settings on users `project` path.

```js
import projects from './projects.acl.js'

export default {
  'fields': {
    '*': {
      'ALLOW': {
        'ADMIN': 'CRUD'
      }
    },
    'projects': {
      'ALLOW': {
        'OWNER': 'U'
      },
      'ref': projects
    }
  }
}
```

### Asserts

At the moment, there is only support for an `owner` assertion but more are
planned to come (and for sure you can register your own plugins). Assertions
might be configured through the ACL settings file but some (especially those
working on resource level) might just accept global parameters.

> Sets a single option for the `owner` assert

```js
export default {
  'asserts': {
    'owner': '_id'
  },
  …
}
```

## Auth

This plugin provides a authentication and authorization service on top of
[passportjs](http://passportjs.org/).

> The `auth` middleware

```js
app.get(
  '/users/:id',
  api()
    .auth({collection: User})
    …
```

## Data

You can use this to return custom any data as response to the client without
the need of using the REST middleware.

> The `data` middleware

```js
app.get(
  '/data',
  api()
    .data({'Hello': 'World'})
    .json()
```

## JSON

You can use this middleware to return data in `res.body` as JSON to the client.

> Return a specific user as JSON object

```js
app.get(
    '/users/:id',
    api()
      .rest({collection: User})
      .json()
  )
```

## json:api

This plugin is a basic implementation of the [json:api](http://jsonapi.org/)
convention and returns your RESTful data in a more structured way.

> Returns a user in `json:api` format

```js
app.get(
    '/users/:id',
    api()
      .rest({collection: User})
      .jsonapi({collection: User})
```

At the moment, `links` and `data` properties are supported.

> Example `json:api` response
 
```JSON
{
  "links": {
    "self": "http://example.com/posts",
    "next": "http://example.com/posts?page[offset]=2",
    "last": "http://example.com/posts?page[offset]=10"
  },
  "data": [{
    "type": "posts",
    "id": "1",
    "attributes": {
      "title": "JSON API paints my bikeshed!"
    }
  ]
}
```

## Map

<aside class="notice">
Consider the nested syntax as deprecated. This will change in the future.
</aside>

> The `map` middleware

```js
app.get(
    '/users',
    api()
      .rest({collection: User})
      .map.res({_id: 'id'})
      .json()
```

This is a very useful plugin to transform output, based on simple mapping rules.
Think of `_id` field returned by [MongoDB](https://www.mongodb.org/) documents.
There is quite a chance that you want to transform it into `id`.

This settings would transform all occurences of the `_id` field from `_id ` to `id`. 

```js
app.get(
  '/users',
  api()
    .rest({collection: User})
    .map.res({_id: 'id'}, {expand: {campaigns: {_id: 'id'}}})
    .json()
)
```

### Nested

This plugin does also support nested mapping via the `expand` parameter.

It will also transform the nested `_id` fields if the `campaigns` path has been
expanded.

## Merge

<aside class="warning">
Use with care. This might result in dozens or hundreds of queries. It is also
prone to race conditions as MongoDB does not support transactions.
</aside>

This is an advanced feature that should be used with care. The `merge` plugin
allows you to really `merge` two referenced documents together.

> Example route

```js
app.get(
  '/users/:user/games',
  api()
    .rest({collection: UserGame})
    .merge({
      collection: Game,
      refField: 'game'
    })
    .json()
)
```

> Example response

```js
[
  {
    "score": 1230550, // <- this is from the UserGame resource 
    "name": "my-super-game" // <- this from the Game resource
  }
]
```

### Example

Think of a document of type `game`. Every user that is playing the game wants
to save the score to the specific game object, but as an API developer you know
that you should separate `state` from `structure`. As the game is played as
app on a mobile phone, you want to give the app developer all the data within
one request.

Merge precedence is original resource over merged resource. In this case and if
you have two `_id` fields that would be merged, the `UserGame` `_id` would be
the one in the response object.

## REST

<aside class="warning">
The <code>rest</code> plugin depends on MongoDB and Mongoose at the moment.
The plan for this library  is to be independent from the storage engine and type
of database. Consider everything returned by this plugin as plain JavaScript
object.
</aside>

> Basic usage of `rest` middleware

```js
api()
  .rest({collection: User})
```

The `rest` plugin is a `CRUD` implementation **close** the REST specification.
It checks the HTTP verb used for a request and calls the right function on the
input or output data.
 
| Verb     | Endpoint	      | Action                              |
|---------:|---	              |---	                                |
| `GET`    | */documents/:id* | fetch a single document  	        |
| `GET`    | */documents*  	  | fetch all documents from endpoint   |
| `POST`   | */documents*  	  | create a new document           	|
| `PUT`    | */documents/:id* | update an existing document     	|
| `DELETE` | */documents/:id* | delete a specific document          |

### On subresources

```js
app.get(
    '/users/:user/projects',
    api()
      .rest({
        collection: Projects
        map: {'owner': ':user'}
      })
      .json()
```

The plugin does also work on subresources. You only have to map the `route`
parameter to a field which identifies the subresource as child of the resource.

In this example, the projects collection is expected to have an `ownership` field
which must be set to the same value as the route parameter `:user`.

### One-to-many and many-to-one relationships

<aside class="notice">
The usage of the words <code>ONE_TO_MANY</code> and <code>MANY_TO_ONE</code> is
ambiguous and will be fixed in a future release.
</aside>

This example is also a good example for a one-to-many relationship. One user can
have many projects.

```js
app.get(
    '/users/:user/projects',
    api()
      .rest({
        collection: User,
        relationship: Resource.MANY_TO_ONE,
        relationshipField: 'projects',
        relationshipCollection: Project,
        map: {'_id': ':user'}
      })
      .json()
  )
```

It might also happen, that you don't want to store the relationship in every
child but you might just wanna maintain an array with documents in your main
resource. The `rest` plugin allows you to handle this scenario too.

## Transform

```s
app.get(
  '/users',
  api()
    .rest({collection: User})
    .transform((req, res) => {
      res.body = res.body.map(niceMapFunction)
    })
    .json()
```

You can use this plugin to apply custom transformation on your response. This
is also a good starting point for developing your custom plugin logic as it
can be used to test program logic on actual output.

## But, there is more…

There are more plugins in the library but those are not ready to use or might
be removed in the future.