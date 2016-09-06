---
layout: middleware
title: Express csurf middleware
menu: resources
lang: en
redirect_from: '/resources/middleware/csurf.html'
name: csurf
---
<div id="page-doc" markdown="1">
# csurf

[![NPM Version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]
[![Build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]
[![Gratipay][gratipay-image]][gratipay-url]

Node.js [CSRF](https://en.wikipedia.org/wiki/Cross-site_request_forgery) protection middleware.

Requires either a session middleware or [cookie-parser](https://www.npmjs.com/package/cookie-parser) to be initialized first.

  * If you are setting the ["cookie" option](#cookie) to a non-`false` value,
    then you must use [cookie-parser](https://www.npmjs.com/package/cookie-parser)
    before this module.
  * Otherwise, you must use a session middleware before this module. For example:
    - [express-session](https://www.npmjs.com/package/express-session)
    - [cookie-session](https://www.npmjs.com/package/cookie-session)

If you have questions on how this module is implemented, please read
[Understanding CSRF](https://github.com/pillarjs/understanding-csrf).

## Installation

```sh
$ npm install csurf
```

## API

```js
var csurf = require('csurf')
```

### csurf([options])

Create a middleware for CSRF token creation and validation. This middleware
adds a `req.csrfToken()` function to make a token which should be added to
requests which mutate state, within a hidden form field, query-string etc.
This token is validated against the visitor's session or csrf cookie.

#### Options

The `csurf` function takes an optional `options` object that may contain
any of the following keys:

##### cookie

Determines if the token secret for the user should be stored in a cookie
or in `req.session`. Defaults to `false`.

When set to `true` (or an object of options for the cookie), then the module
changes behavior and no longer uses `req.session`. This means you _are no
longer required to use a session middleware_. Instead, you do need to use the
[cookie-parser](https://www.npmjs.com/package/cookie-parser) middleware in
your app before this middleware.

When set to an object, cookie storage of the secret is enabled and the
object contains options for this functionality (when set to `true`, the
defaults for the options are used). The options may contain any of the
following keys:

  - `key` - the name of the cookie to use to store the token secret
    (defaults to `'_csrf'`).
  - `path` - the path of the cookie (defaults to `'/'`).
  - any other [res.cookie](http://expressjs.com/4x/api.html#res.cookie)
    option can be set.

##### ignoreMethods

An array of the methods for which CSRF token checking will disabled.
Defaults to `['GET', 'HEAD', 'OPTIONS']`.

##### sessionKey

Determines what property ("key") on `req` the session object is located.
Defaults to `'session'` (i.e. looks at `req.session`). The CSRF secret
from this library is stored and read as `req[sessionKey].csrfSecret`.

If the ["cookie" option](#cookie) is not `false`, then this option does
nothing.

##### value

Provide a function that the middleware will invoke to read the token from
the request for validation. The function is called as `value(req)` and is
expected to return the token as a string.

The default value is a function that reads the token from the following
locations, in order:

  - `req.body._csrf` - typically generated by the `body-parser` module.
  - `req.query._csrf` - a built-in from Express.js to read from the URL
    query string.
  - `req.headers['csrf-token']` - the `CSRF-Token` HTTP request header.
  - `req.headers['xsrf-token']` - the `XSRF-Token` HTTP request header.
  - `req.headers['x-csrf-token']` - the `X-CSRF-Token` HTTP request header.
  - `req.headers['x-xsrf-token']` - the `X-XSRF-Token` HTTP request header.

## Example

### Simple express example

The following is an example of some server-side code that generates a form
that requires a CSRF token to post back.

```js
var cookieParser = require('cookie-parser')
var csrf = require('csurf')
var bodyParser = require('body-parser')
var express = require('express')

// setup route middlewares
var csrfProtection = csrf({ cookie: true })
var parseForm = bodyParser.urlencoded({ extended: false })

// create express app
var app = express()

// parse cookies
// we need this because "cookie" is true in csrfProtection
app.use(cookieParser())

app.get('/form', csrfProtection, function(req, res) {
  // pass the csrfToken to the view
  res.render('send', { csrfToken: req.csrfToken() })
})

app.post('/process', parseForm, csrfProtection, function(req, res) {
  res.send('data is being processed')
})
```

Inside the view (depending on your template language; handlebars-style
is demonstrated here), set the `csrfToken` value as the value of a hidden
input field named `_csrf`:

```html
<form action="/process" method="POST">
  <input type="hidden" name="_csrf" value="{{csrfToken}}">

  Favorite color: <input type="text" name="favoriteColor">
  <button type="submit">Submit</button>
</form>
```

### Custom error handling

When the CSRF token validation fails, an error is thrown that has
`err.code === 'EBADCSRFTOKEN'`. This can be used to display custom
error messages.

```js
var bodyParser = require('body-parser')
var cookieParser = require('cookie-parser')
var csrf = require('csurf')
var express = require('express')

var app = express()
app.use(bodyParser.urlencoded({ extended: false }))
app.use(cookieParser())
app.use(csrf({ cookie: true }))

// error handler
app.use(function (err, req, res, next) {
  if (err.code !== 'EBADCSRFTOKEN') return next(err)

  // handle CSRF token errors here
  res.status(403)
  res.send('form tampered with')
})
```

## License

[MIT](LICENSE)

[npm-image]: https://img.shields.io/npm/v/csurf.svg
[npm-url]: https://npmjs.org/package/csurf
[travis-image]: https://img.shields.io/travis/expressjs/csurf/master.svg
[travis-url]: https://travis-ci.org/expressjs/csurf
[coveralls-image]: https://img.shields.io/coveralls/expressjs/csurf/master.svg
[coveralls-url]: https://coveralls.io/r/expressjs/csurf?branch=master
[downloads-image]: https://img.shields.io/npm/dm/csurf.svg
[downloads-url]: https://npmjs.org/package/csurf
[gratipay-image]: https://img.shields.io/gratipay/dougwilson.svg
[gratipay-url]: https://gratipay.com/dougwilson/
</div>