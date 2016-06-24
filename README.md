# Errors

A set of error classes and [Express](http://expressjs.com/) error handlers to be used in a REST service.

**NOTE**: this library is not very customizable nor will it be, its intent is to serve as a standard for my projects.

# Install

```
$ npm install --save @nielskrijger/errors
```

# Usage

## Errors

You can use the provided error class like this:

```js
import {
  ServerError,
  NotFoundError,
  BadRequestError,
  ForbiddenError,
  ValidationError,
  ValidationErrors,
} from '@nielskrijger/errors';

// { httpStatus: 400, code: 'bad_request', message: 'One or more validation errors were found' }
throw new BadRequestError();

// { httpStatus: 403, code: 'forbidden', message: 'Insufficient privileges' }
throw new ForbiddenError();

// { httpStatus: 404, code: 'not_found', message: 'Resource not found' }
throw new NotFoundError();

// { httpStatus: 500, code: 'server_error', message: 'An internal server problem occurred' }
throw new ServerError();

/*
{
  httpStatus: 400,
  code: 'bad_request',
  message: 'One or more validation errors were found',
  errors: [{ path: '/metric_id', code: 'not_found', message: 'metric not found' }],
}
*/
throw new ValidationError('/metric_id', 'not_found', 'metric not found');
throw new ValidationErrors([
  { path: '/name', code: 'min_length', message: 'must have at least 5 characters' },
  { path: '/description', code: 'required', message: 'is required' },
]);
```

## Express error handlers

The express error handlers part of this package transform `RestError`s to the following error response format:

```json
{
  "error": "not_found",
  "error_description": "Resource not found",
  "error_details": [{
    "code": "min_length",
    "path": "/name",
    "message": "must have at least 5 characters"
  }]
}
```

This format is similar to the [OAuth2 Error syntax](https://tools.ietf.org/html/rfc6749#section-4.1.2.1).

You can use error handlers like this:

```js
import express from 'express';
import log from '@nielskrijger/logger';
import {
  ServerError,
  restErrorHandler,
  unexpectedErrorHandler,
} from '@nielskrijger/errors';

log.init();

var app = express();

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.get('/validation_error', (req, res) => {
  throw new ValidationErrors([
    { path: '/name', code: 'min_length', message: 'must have at least 5 characters' },
    { path: '/description', code: 'required', message: 'is required' },
  ]);
});

app.get('/unexpected_error', (req, res) => {
  throw new Error('Something happened');
});

app.use(restErrorHandler(log));
app.use(unexpectedErrorHandler(log));

app.listen(3000, () => {
  console.log('Example app listening on port 3000!');
});
```
