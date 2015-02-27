Traverson API
=============

This is the reference documenation for the Traverson API. It is rather technical and concise. Also see the [readme](https://github.com/basti1302/traverson/blob/master/readme.markdown), which explains the concepts behind Traverson in greater depths and has a lot of examples for the various features.

Traverson
---------

Traverson is all about link traversal. It is intended to be used with Hypermedia APIs, APIs that are composed of representations of linked resources. Traverson uses the link relations between the resources to find its way to the target resource. Without Traverson, your API client would have to make an HTTP request to each resource, look for the link relation you want to follow and make another HTTP request to the link behind that link relation, multiple times until you find your target resource. With Traverson, you only specify the link relations you want to follow and Traverson executes all required HTTP requests and handles the process of following the link relations. This process is called the *link traversal process* in the remainder of this document and it is a central concept of Traverson.

### Methods

This methods are available on the Traverson object - the object acquired by `var traverson = require('traverson')` or the global `traverson` object when using the browser build with a script tag.

`newRequest()`: Returns a new [request builder](#request-builder) instance.

`from(url)`: Returns a new [request builder](#request-builder) instance with the given root URL. Thus, `traverson.from(url)` is equivalent to newRequest().from(url)`.

`registerMediaType(contentType, constructor)`: Registers a new media type plug-in. `contentType` should be the RFC2046 media type and `constructor` is ought to be a constructor function that can be used to create new instances of the plug-in.

### Properties

This properties are available on the Traverson object - the object acquired by `var traverson = require('traverson')` or the global `traverson` object when using the browser build with a script tag.

`mediaTypes`: A map of media types that can be used with `setMediaType` (see below).

`json`: *Deprecated* An object that only has one method, `from(url)`, which in turn creates a new [request builder](#request-builder). Use `traverson.from(url).json()` instead of `traverson.json.from(url)`. Or use `traverson.from(url)` and let Traverson figure out the media type by using the Content-Type header send by the server.

`jsonHal`: *Deprecated* An object that only has one method, `from(url)`, which in turn creates a new [request builder](#request-builder). Use `traverson.from(url).jsonHal()` instead of `traverson.jsonHal.from(url)`. Or use `traverson.from(url)` and let Traverson figure out the media type by using the Content-Type header send by the server. The media type plug-in `traverson-hal` has to be installed for HAL support as of version 1.0.0.

Request Builder
---------------

A request builder can be obtained by `traverson.newRequest()` or `traverson.from(url)`. It is used to prepare and execute a single link traversal process. The request builder offers two types of methods: configuration methods and action methods. You can call any number of configuration methods on request builder instance to prepare the link traversal process. You can also chain configuration method calls because they return the request builder instance. When you are done configuring it is time to call one of the action methods. They tell Traverson what to do at the end of the link traversal process. In contrast to the configuration methods you must only call one of the action methods on any request builder instance and should not call a configuration methods after you have called an action method.

### Configuration Methods

`setMediaType(mediaType)`: Disables content negotiation and forces Traverson to assume the given media type when parsing and interpreting the response bodies from the server. This method returns the request builder instance to allow for method chaining. The `mediaType` parameter should be a registered media type, like `application/json`, `application/hal+json`, and so on. Some media types are available as constants via the object `traverson.mediaTypes`. A media type plug-in needs to be registered for all media types except `application/json`.

`json()`: Shortcut for `setMediaType(application/json)`. Disables content negotiation and forces Traverson to assume the media type `application/json` when parsing and interpreting the response bodies from the server. Returns the request builder instance to allow for method chaining.

`jsonHal()`: Shortcut for `setMediaType(application/hal+json)`. Disables content negotiation and forces Traverson to assume the media type `application/hal+json` when parsing and interpreting the response bodies from the server. The media type plug-in `traverson-hal` has to be installed for HAL support as of version 1.0.0. This method returns the request builder instance to allow for method chaining.

`useContentNegotiation()`: Enables content negotiation, that is, the server's response bodies are parsed and interpreted according to the Content-Type header. This is the default behaviour. Calling `useContentNegotiation` reverses the effect of a former call to `setMediaType`, `json` or `jsonHal`. This method returns the request builder instance to allow for method chaining.

`from(url)`: Set the root URL of the API, that is, where the link traversal begins. If you created the request builder instance with `traverson.from(url)` you don't need to call `from` on the request builder instance. This method returns the request builder instance to allow for method chaining.

`follow(links)`: Provides the list of link relations to follow. Returns the request builder instance to allow for method chaining.

`walk(links)`: An alias for `follow`. Returns the request builder instance to allow for method chaining.

`withTemplateParameters(parameters)`: Provide template parameters for URI template substitution. Returns the request builder instance to allow for method chaining.

`withRequestOptions(options)`: Provide options for HTTP requests (additional HTTP headers, for example). Returns the request builder instance to allow for method chaining.

`withRequestLibrary(request)`: Injects a custom request library. Returns the request builder instance to allow for method chaining.

`parseResponseBodiesWith(parser)`: Injects a custom JSON parser. Returns the request builder instance to allow for method chaining.

`resolveRelative()`: Switches URL resolution to relative (default is absolute). This is for relative URL paths, that is, URLs that omit the protocol (http/https), but start with a slash and that need to be interpreted relative to the current location. Example: If the root URL is `https://api.example.com/home` and the first link contains `/customers/1302` this would usually (without `resolveRelative()`) be resolved to `https://api.example.com/customers/1302`. If this has a link `/orders`, this would be resolved to `https://api.example.com/orders`. When `resolveRelative()` has been called on the request builder instance, the URLs will be resolved differently: From `https://api.example.com/home` the link `/customers/1302` will be resolved to `https://api.example.com/home/customers/1302`. From there, the link `/orders` will be resolved to `https://api.example.com/home/customers/1302/orders`. This feature should be rarely needed. This method returns the request builder instance to allow for method chaining.

`newRequest()`: Returns a clone of the request builder with the same configuration. This method can be called before or after any of the action methods. All configuration options that have been set on the original request builder will also be set on the returned instance, with the exception of the parameter(s) given to the `follow` method which are not copied to the new instance. Also, if an action method has called before calling `newRequest()` on the original request builder, no state from the execution of the action method will be known to the new request builder instance.

### Action Methods

Calling one of this methods finishes the configuration phase and starts the link traversal process. The link traversal process works exactly the same for each of the action methods, with the exception of the last request and how the response of the last response is processed before handing it back to the callback.

Exactly one of this methods should be called after calling zero or more configuration methods. Once one of these methods has been called, the request builder instance should not be used anymore.

`getResource(callback)`: This method is what you probably want to call if you want to retrieve information from the remote API. It will parse the JSON body from the last HTTP response and pass the resulting JavaScript object to your callback. The callback signature is `callback(err, resource)`.

`get(callback)`: This method is similar to `getResource`, but it does not parse the HTTP response body for you. Instead, it gives you the response object, including the HTTP status code and the raw body. The callback signature is `callback(err, response)`.

`getUri(callback)`: This method is similar to `getResource` and `get`, but it will actually not execute the last HTTP request in the link traversal process. Instead it will pass the URL it has discovered for the last HTTP request back to the client so that the client can execute the HTTP request itself. The callback signature is `callback(err, url)`.

`post(body, callback)`: Instead of sending a GET request to the last URL in the link traversal process, Traverson will send a POST request with the given `body`. The callback signature is `callback(err, response, url)`.

`put(body, callback)`: Instead of sending a GET request to the last URL in the link traversal process, Traverson will send a PUT request with the given `body`. The callback signature is `callback(err, response, url)`.

`patch(body, callback)`: Instead of sending a GET request to the last URL in the link traversal process, Traverson will send a PATCH request with the given `body`. The callback signature is `callback(err, response, url)`.

`delete(callback)`: Instead of sending a GET request to the last URL in the link traversal process, Traverson will send a DELETE request. The callback signature is `callback(err, response, url)`.

`del(callback)`: An alias for `delete`.