# http-authenticator ![CI](https://github.com/jambonz/http-authenticator/workflows/CI/badge.svg)

drachtio middleware that delegates sip authentication to an http api.  This allows, for instance, a multi-tenant sip application server to delegate authentication to a customer api.

The middleware-returning function can be invoked EITHER with an HTTP URL to call OR a function yielding a Promise that resolves to an HTTP URL.  Alternatively, instead of a returning a URL, and object can be provided with `url`, `username`, and `password` properties if you wish to protect your endpoint with HTTP Basic Authentication. 

An HTTP POST will be made to the specified URL with a JSON body containing the sip method and the components from the Authorization header.  The HTTP server should return a status code of 200 in all cases, containing a JSON body with instructions on whether to admit the request.

To admit the request, send a 200 response with a `status` of `ok`, e.g.
```
{"status": "ok"}
```
To deny the request, send a 200 response with a `status` of `fail`.  The `status` field MUST be provided.  Optionally, a response MAY include a `msg` attribute, an `expires` attribute, and/or a `blacklist` attribute.  

- The `msg` property is simply a human-readable description of why an authentication failed.
- The `expires` value provides a value in seconds for the duration of a granted registration.  This value, if provided, must be less than the requested expiration. If not provided, the requested expires value is granted.
- The `blacklist` property shall contain a number indicating a period of time, in seconds, that the source IP address should be blocked.  A value of -1 means forever.
```
{"status": "fail"}
```
or
```
{"status": "fail", "msg": "unknown user"}
```
```
{"status": "fail", "blaclist": 3600}
```


Additionally, for admitted requests, the middleware adds a `req.authorization` object which contains two properties:
- challengeResponse - an object containing the parsed elements of the sip Authorization header, and
- grant - an object containing the json response received in the 200 OK to the POST request.

```
const authenticator = require('drachtio-http-authenticator')({
  url: 'https://example.com/auth',
  auth: {
    username: 'foo',
    password: 'bar'
  }
});

srf.use('invite', authenticator);
```