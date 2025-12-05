# Java Http Server

My next app ([this](https://mijnwoonplaats.gjk101.com/) is the previous one, here is the GitHub [repo](https://github.com/GeertJan-Kuip/mijnwoonplaats)) will again be a simple website with just one funcionallity. It lets you copy-paste the url of a GitHub page. If this page shows the contents of a Java package, the code of the package will be processed and turned into a uml schema, which will be displayed. The processing is done based on AST (Abstract Syntax Tree) files which are created with javac libraries. To display the schema, Mermaid is being udes (see previous blog).

Instead of using Spring or Spring Boot I will opt for a very basic and lightweight variant. ChatGPT suggested the in-built `com.sun.net.httpserver` package which can do nothing special but is very light and is part of the JDK. I might use Jackson for some JSON parsing and some Maven plugin to create a fat jar but that would  be it. The app will be more like 1Mb instead of 30Mb, testing will be faster, no application.properties is required and the pom-file will be modest in size. It will run in a Docker container and I think I will create a CI/CD pipeline using GitHub Actions, similar to the previous project.

## com.sun.net.httpserver

The FQN of the package shows the legacy-like character of this webserver, according to the [documentation](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.httpserver/com/sun/net/httpserver/package-summary.html) it was introduced in version 6, which was in 2006. There seems to be support for basic authentication, which I do not need. As I suppose that Nginx will take care of the https implementation I suppose the most relevant classes are HttpServer, HttpExchange and HttpContext. 

### HttpServer

Documentation of this class is [here](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.httpserver/com/sun/net/httpserver/HttpServer.html). This object must be created with the static factory method `create(..)` that is in the class itself. You can supply a port number using `new InetSocketAddress(port)` as first argument. The `createContext()` method creates a HttpContext object which is responsible for the mapping of requests to response actions. The server is started with the `start()` method, upon which it starts listening via the provided port.

### HttpExchange

Documentation [here](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.httpserver/com/sun/net/httpserver/HttpExchange.html), it says: _This class encapsulates a HTTP request received and a response to be generated in one exchange. It provides methods for examining the request from the client, and for building and sending the response._

The documentation provides a description of the typical use of HttpExchange:

- getRequestMethod() to determine the command
- getRequestHeaders() to examine the request headers (if needed)
- getRequestBody() returns a InputStream for reading the request body. After reading the request body, the stream is close.
- getResponseHeaders() to set any response headers, except content-length
- sendResponseHeaders(int,long) to send the response headers. Must be called before next step.
- getResponseBody() to get a OutputStream to send the response body. When the response body has been written, the stream must be closed to terminate the exchange.

If this has been done, call close() as a last step. What surprises me is that methods involved in sending responses start with 'get', it suggests that they have been available already in the HttpExchange object but need additions. `getResponseHeaders()` and `sendResponseHeaders()` are complementary. `getResponseBody()` returns an OutputStream object to which the response must be written in the form of a `byte[]`  (bytearray), like this:

```
exchange.getResponseBody().write(stringvariable.getBytes());
```

What is easy about all this is that all content related to request and response lives in the same object.

### HttpContext

This object contains 



