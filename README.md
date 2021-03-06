# Express JWT Permissions

[![Build Status](https://travis-ci.org/MichielDeMey/express-jwt-permissions.svg?branch=master)](https://travis-ci.org/MichielDeMey/express-jwt-permissions)
[![Coverage Status](https://coveralls.io/repos/MichielDeMey/express-jwt-permissions/badge.svg?branch=master&service=github)](https://coveralls.io/github/MichielDeMey/express-jwt-permissions?branch=master)
[![npm](https://img.shields.io/npm/dm/express-jwt-permissions.svg?maxAge=2592000)](https://www.npmjs.com/package/express-jwt-permissions)

[![js-standard-style](https://cdn.rawgit.com/feross/standard/master/badge.svg)](https://github.com/feross/standard)

Middleware that checks JWT tokens for permissions, recommended to be used in conjunction with [express-jwt](https://github.com/auth0/express-jwt).

## Install

```
npm install express-jwt-permissions --save
```

## Usage

This middleware assumes you already have a JWT authentication middleware such as [express-jwt](https://github.com/auth0/express-jwt).

The middleware will check a decoded JWT token to see if a token has permissions to make a certain request.

Permissions should be described as an array of strings inside the JWT token, or as a space-delimited [OAuth 2.0 Access Token Scope](https://tools.ietf.org/html/rfc6749#section-3.3) string.

```json
"permissions": [
  "status",
  "user:read",
  "user:write"
]
```

```json
"scope": "status user:read user:write"
```

If your JWT structure looks different you should map or reduce the results to produce a simple Array or String of permissions.

### Using permission Array
To verify a permission for all routes using an array:

```javascript
var guard = require('express-jwt-permissions')()

app.use(guard.check('admin'))
```

If you require different permissions per route, you can set the middleware per route.

```javascript
var guard = require('express-jwt-permissions')()

app.get('/status', guard.check('status'), function(req, res) { ... })
app.get('/user', guard.check(['user:read']), function(req, res) { ... })
```

### Matching segmented permissions
Every entry in a required array must be matched on protected paths.  However,
each entry can be configured to match multiple segmented permissions.

In order for segmented permissions to match, the number of segments in each must
be the same and for each segment there must be at least 1 match among the comma
separated values.

```javascript
var guard = require('express-jwt-permissions')()
// req = {
//   user: {
//     permissions: [
//       'requests:edit:moderator'
//     ]
//   }
// }

// Match any of requests:edit:owner requests:edit:admin requests:edit:moderator
app.put('/requests/{rid}/thing', guard.check('requests:edit:owner,admin,moderator'), function(req, res) { ... })
```

The reverse is also true.

```javascript
// req = {
//   user: {
//     permissions: [
//       'requests:edit:moderator,admin'
//     ]
//   }
// }

app.put('/requests/{rid}/thing', guard.check('requests:edit:admin'), function(req, res) { ... })
```

### Configuration
To set where the module can find the user property (default `req.user`) you can set the `requestProperty` option.

To set where the module can find the permissions property inside the `requestProperty` object (default `permissions`), set the `permissionsProperty` option.

Example:

Consider you've set your permissions as `scope` on `req.identity`, your JWT structure looks like:

```json
"scope": "user:read user:write"
```

You can pass the configuration into the module:

```javascript
var guard = require('express-jwt-permissions')({
  requestProperty: 'identity',
  permissionsProperty: 'scope'
})

app.use(guard.check('user:read'))
```

## Error handling

The default behavior is to throw an error when the token is invalid, so you can add your custom logic to manage unauthorized access as follows:

```javascript
app.use(guard.check('admin'))

app.use(function (err, req, res, next) {
  if (err.code === 'permission_denied') {
    res.status(401).send('insufficient permissions');
  }
});
```

**Note** that your error handling middleware should be defined after the jwt-permissions middleware.

## Tests

    $ npm install
    $ npm test

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE.txt) file for more info.
