# pogo [![Build status for Pogo](https://travis-ci.com/sholladay/pogo.svg?branch=master "Build Status")](https://travis-ci.com/sholladay/pogo "Builds")

> Server framework for [Deno](https://github.com/denoland/deno)

Pogo is an easy to use, safe, and expressive framework for writing web servers and applications. It is inspired by [hapi](https://github.com/hapijs/hapi).

## Contents

 - [Why?](#why)
 - [Usage](#usage)
 - [API](#api)
 - [Contributing](#contributing)
 - [License](#license)

## Why?

 - Designed to encourage reliable and testable applications.
 - Routes are simple, pure functions that return values directly.
 - Automatic JSON responses from objects.

## Usage

```js
import pogo from 'https://deno.land/x/pogo/main.js';

const app = pogo.server({ port : 3000 });

app.route({
    method : 'GET',
    path   : '/hello',
    handler() {
        return 'Hello, world!';
    }
});

app.start();
```

## API

 - [`server = pogo.server(option)`](#server--pogoserveroption)
 - [`server.route(option)`](#serverrouteoption)
 - [`server.start()`](#serverstart)
 - [Response Toolkit](#response-toolkit)
   - [`h.body(body)`](#hbodybody)
   - [`h.code(statusCode)`](#hcodestatuscode)
   - [`h.created(url)`](#hcreatedurl)
   - [`h.header(name, value)`](#hheadername-value)
   - [`h.location(url)`](#hlocationurl)
   - [`h.permanent()`](#hpermanent)
   - [`h.redirect(url)`](#hredirecturl)
   - [`h.rewritable(isRewritable)`](#hrewritableisrewritable)
   - [`h.temporary()`](#htemporary)
   - [`h.type(mediaType)`](#htypemediatype)

### server = pogo.server(option)

Returns a server instance, which can then be used to add routes and start listening for requests.

#### option

Type: `object`

##### hostname

Type: `string`<br>
Default: `localhost`

Specifies which domain or IP address the server will listen on when `server.start()` is called. Use `0.0.0.0` to listen on any hostname.

##### port

Type: `number`<br>
Example: `3000`

Specifies which port number the server will listen on when `server.start()` is called. Use `0` to listen on any available port.

### server.route(option)

Adds a route to the server so that the server knows how to respond to requests for the given HTTP method and path, etc.

#### option

Type: `object`

##### method

Type: `string`<br>
Example: `GET`

Any valid HTTP method. Used to limit which requests will trigger the route handler.

##### path

Type: `string`<br>
Example: `/`

Any valid URL path. Used to limit which requests will trigger the route handler.

##### handler(request, h)

Type: `function`

 - `request` is a [ServerRequest](https://github.com/denoland/deno_std/blob/e28c9a407951f10d952993ff6a7b248ca11243e1/http/http.ts#L123-L274) instance with properties for `headers`, `method`, `url`, and more.
 - `h` is a [Response Toolkit](#response-toolkit) instance, which has utility methods for modifying the response.

The implementation for the route that handles requests. Called when a request is received that matches the `method` and `path` specified in the route options.

Should return one of:
 - A `string`, which will be sent as HTML.
 - A JSON value, which will be sent as JSON using [`JSON.stringify()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).
 - A [Response Toolkit](#response-toolkit) instance, which will send the `h.body()` value, if any.

An appropriate [`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) header will be set automatically based on the response body before the response is sent. You can use [`h.type()`](#htypemediatype) to override the default behavior.

### server.start()

Begins listening on the `hostname` and `port` specified when the server was created.

### Response Toolkit

The response toolkit is an object that is passed to route handlers, with utility methods that make it easy to modify the response. For example, you can use it to set headers or a status code.

By convention, this object is assigned to a variable named `h` in code examples.

#### h.body(body)

Sets the response body. This is the same as returning the body directly from the route handler, but it's useful in order to begin a chain with other toolkit methods.

#### h.code(statusCode)

Sets the response status code. When possible, it is better to use a more specific method instead, such as [`h.redirect()`](#hredirecturl).

*Tip: Use Deno's `status` constants to define the status code.*

```js
import { Status as status } from 'https://deno.land/x/http/http_status.ts';
const handler = (request, h) => {
    return h.code(status.Teapot);
};
```

#### h.created(url)

Sets the response status to [`201 Created`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/201) and sets the [`Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location) header to the value of `url`.

Returns the toolkit so other methods can be chained.

#### h.header(name, value)

Sets a response header. Always replaces any existing header of the same name. Headers are case insensitive.

Returns the toolkit so other methods can be chained.

#### h.location(url)

Sets the [`Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location) header on the response to the value of `url`. When possible, it is better to use a more specific method instead, such as [`h.created()`](#hcreatedurl) or [`h.redirect()`](#hredirecturl).

Returns the toolkit so other methods can be chained.

#### h.permanent()

*Only available after calling the h.redirect() method.*

Sets the response status to [`301 Moved Permanently`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/301) or [`308 Permanent Redirect`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/308) based on whether the existing status is considered rewritable (see "method handling" on [Redirections in HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections) for details).

Returns the toolkit so other methods can be chained.

#### h.redirect(url)

Sets the response status to [`302 Found`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/302) and sets the [`Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location) header to the value of `url`.

Also causes some new toolkit methods to become available for customizing the redirect behavior:

 - [`h.permanent()`](#hpermanent)
 - [`h.temporary()`](#htemporary)
 - [`h.rewritable()`](#hrewritableisrewritable)

Returns the toolkit so other methods can be chained.

#### h.rewritable(isRewritable)

*Only available after calling the h.redirect() method.*

Sets the response status to [`301 Moved Permanently`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/301) or [`302 Found`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/302) based on whether the existing status is a permanent or temporary redirect code. If `isRewritable` is `false`, then the response status will be set to [`307 Temporary Redirect`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/307) or [`308 Permanent Redirect`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/308).

Returns the toolkit so other methods can be chained.

#### h.temporary()

*Only available after calling the h.redirect() method.*

Sets the response status to [`302 Found`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/302) or [`307 Temporary Redirect`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/307) based on whether the existing status is considered rewritable (see "method handling" on [Redirections in HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections) for details).

Returns the toolkit so other methods can be chained.

#### h.type(mediaType)

Sets the [`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) header on the response to the value of `mediaType`.

Overrides the media type that is set automatically by the framework.

Returns the toolkit so other methods can be chained.

## Contributing

See our [contributing guidelines](https://github.com/sholladay/pogo/blob/master/CONTRIBUTING.md "Guidelines for participating in this project") for more details.

1. [Fork it](https://github.com/sholladay/pogo/fork).
2. Make a feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. [Submit a pull request](https://github.com/sholladay/pogo/compare "Submit code to this project for review").

## License

[MPL-2.0](https://github.com/sholladay/pogo/blob/master/LICENSE "License for pogo") © [Seth Holladay](https://seth-holladay.com "Author of pogo")

Go make something, dang it.
