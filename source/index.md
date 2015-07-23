---
title: API Reference

language_tabs:
  - javascript

toc_footers:
  - <a href='https://neti.am'>Project Website</a>
  - <a href='https://github.com/netiam/netiam'>Github Repository</a>

includes:
  - errors

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

```bash
npm i -S netiam
```

# Routes

```js
export default function(app) {
  let router = express.Router()

  router.get('/', function(req, res) {
    res.redirect('/v1/docs')
  })

  users(router)

  router.use('/docs', express.static(__dirname + '/../public/docs'))

  app.use('/v1', router)
  app.get('/', function(req, res) {
    res.redirect('/v1/docs')
  })
}
```

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

# Plugins

## ACL

## Auth

## Cache

## Client

## Data

## Graph

## JSON

## json:api

## Map

## Merge

## Profile

## Render

## REST

## State

## Transform
