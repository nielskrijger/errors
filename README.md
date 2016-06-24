# Errors

A set of error classes and [Express](http://expressjs.com/) error handlers to be used in a REST service.

**NOTE**: this library is not very customizable nor will it be, its intent is to serve as a standard for my projects.

# Install

```
$ npm install --save @nielskrijger/errors
```

# Usage

## Errors

The following errors classes are available:

Class name        | HTTP | Code         | Error message
------------------|------|--------------|------------------------------------------
ServerError       |  500 | server_error | An internal server problem occurred
BadRequestError   |  400 | bad_request  | One or more validation errors were found
UnauthorizedError |  401 | unauthorized | Invalid credentials
ForbiddenError    |  403 | forbidden    | Insufficient privileges
NotFoundError     |  404 | not_found    | Resource not found
ValidationError   |  400 | bad_request  | One or more validation errors were found
ValidationErrors  |  400 | bad_request  | One or more validation errors were found

**Changing error code/message**

Change the error message as follows:

```js
import {
  BadRequestError,
} from '@nielskrijger/errors';

throw new BadRequestError('invalid_syntax', 'The message body is malformed');
```

**Custom error class**

When changing the error code consider creating a subclass instead:

```js
import { BadRequestError } from '@nielskrijger/errors';

export class SyntaxError extends BadRequestError {
  constructor() {
    super('invalid_syntax', 'The message body is malformed');
  }
}
```

**Validation errors**

Validation errors extend `BadRequestError`s and add additional error details for the client .

```js
import {
  ValidationError,
  ValidationErrors,
} from '@nielskrijger/errors';

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

You can use the error handlers like this:

```js
import express from 'express';
import log from '@nielskrijger/logger';
import {
  ValidationErrors
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
