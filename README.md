domain-middleware [![Build Status](https://secure.travis-ci.org/expressjs/domain-middleware.png)](http://travis-ci.org/expressjs/domain-middleware)
=======

![logo](https://raw.github.com/expressjs/domain-middleware/master/logo.png)

An `uncaughtException` middleware for connect, using `domains` to allow a clean uncaught errors handling. This module tries to be a better [connect-domain](https://github.com/baryshev/connect-domain) module.

Features :
* unit tests, code coverage
* creates a domain for the current request
* detects errors bubbled to this domain
* by default, handles errors with the following "best practices" pattern:
  * responds to the corresponding request with the express error handler (remember to set one !)
  * starts a security suicide timeout (configurable) in case the shutdown blocks
  * if a cluster worker, calls disconnect() (which closes servers and signals the cluster master of our inavailability)
  * if not a cluster, closes the server
* a custom error callback may be provided if those default actions are not suitable
* only starts the error callback once, subsequent errors are printed only
* only closes the server or disconnect the worker if not already done (monitoring begins at the middleware creation)


Tested with express 4. Should work with express 3 and connect.

See also [node-domain-middleware](https://github.com/brianc/node-domain-middleware), [express-domain-errors](https://github.com/mathrawka/express-domain-errors), [express-err](https://github.com/neoziro/express-err)

Interesting reads :
* [Warning: Don't Ignore Errors!](http://nodejs.org/docs/latest/api/domain.html#domain_warning_don_t_ignore_errors)
* [Error Handling in Node.js](http://www.joyent.com/developers/node/design/errors)
* [node.js domain API](http://nodejs.org/api/domain.html)
* [node.js cluster API](http://nodejs.org/api/cluster.html)


## Installation

```bash
$ npm install domain-middleware
```


## Usage

Usually, [domain](http://nodejs.org/api/domain.html) usage goes hand-in-hand with the [cluster](http://nodejs.org/api/cluster.html) module, since the master process can fork a new worker when a worker encounters an error.
Please see [connect_with_cluster](https://github.com/expressjs/domain-middleware/tree/master/example/connect_with_cluster) example.

Note : the example code below throws various errors for test, don't do it in production of course:

```js
var http = require('http');
var express = require('express');
var domainMiddleware = require('domain-middleware');

var app = express();
var server = http.createServer(app);

app.use(domainMiddleware({
  server: server,
  killTimeout: 30000,
}))
.use(function(req, res) {

  if (Math.random() > 0.5) {
    foo.bar(); // synchronous error
  }
  setTimeout(function() {
    if (Math.random() > 0.5) {
      throw new Error('Asynchronous error from timeout');
    } else {
      res.end('Hello from Connect!');
    }
  }, 100);
  setTimeout(function() {
    if (Math.random() > 0.5) {
      throw new Error('Mock second error');
    }
  }, 200);

})
.use(function(err, req, res, next) {
  res.end(err.message);
});


server.listen(1984);
```

If the default error handling (shutdown timeout, server close, cluster disconnect) doesn't fit your need,
a custom error callback can be provided:
```
app.use(domainMiddleware({
  onError: myErrorHandler  <-- params : (req, res, next, err, options)
  // all other options no longer have any meaning unless your custom callback use them
}))
```


## Contributing
Thank you for contributing !

You may want to create an issue first if you are not sure.

* fork
* clone
* `cd domain-middleware`
* `make test`
* (optional : start a branch)
* add tests
* add features
* `make test-cov` and check coverage
* send pull request https://help.github.com/articles/be-social#pull-requests


## License

(The MIT License)

Copyright (c) 2013 fengmk2 &lt;fengmk2@gmail.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
