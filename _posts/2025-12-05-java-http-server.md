# Java Http Server

My next app ([this](https://mijnwoonplaats.gjk101.com/) is the previous one, here is its GitHub [repo](https://github.com/GeertJan-Kuip/mijnwoonplaats)) will again be a simple website with just one functionality. It lets you copy-paste the url of a GitHub page. If this page shows the contents of a Java package, the code of the package will be processed and turned into a uml schema, which will be displayed. The processing is done based on AST (Abstract Syntax Tree) files which are created with javac libraries. To display the schema, Mermaid is being used (see previous blog).

Instead of using Spring or Spring Boot I will opt for a very basic and lightweight variant. ChatGPT suggested the in-built `com.sun.net.httpserver` package which can do nothing special but is very light and is part of the JDK. I might use Jackson for some JSON parsing and some Maven plugin to create a fat jar but that would  be it. The app will be more like 1Mb instead of 30Mb, testing will be faster, no application.properties is required and the pom-file will be modest in size. It will run in a Docker container and I think I will create a CI/CD pipeline using GitHub Actions, similar to the previous project.

## com.sun.net.httpserver

The FQN of the package shows the legacy-like character of this webserver, according to the [documentation](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.httpserver/com/sun/net/httpserver/package-summary.html) it was introduced in version 6, which was in 2006. There seems to be support for basic authentication, which I do not need. As I assume that Nginx will take care of the https implementation I think the most relevant classes are HttpServer, HttpExchange and HttpContext. 

### HttpServer

Documentation of this class is [here](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.httpserver/com/sun/net/httpserver/HttpServer.html). This object must be created with the static factory method `create(..)` that is in the class itself. You can supply a port number using `new InetSocketAddress(port)` as first argument. The `createContext()` method creates a HttpContext object which is responsible for the mapping of requests to response actions. The server is started with the `start()` method, upon which it starts listening via the provided port.

### HttpExchange

Documentation [here](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.httpserver/com/sun/net/httpserver/HttpExchange.html), it says: _This class encapsulates a HTTP request received and a response to be generated in one exchange. It provides methods for examining the request from the client, and for building and sending the response._

The documentation provides a description of the typical order in which you call the methods of HttpExchange:

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

What is easy about all this is that all content related to request and response live in the same object. It is all pretty compact and manageable.

### HttpContext

According to the [docs](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.httpserver/com/sun/net/httpserver/HttpContext.html), _HttpContext represents a mapping between the root URI path of an application to a HttpHandler which is invoked to handle requests destined for that path on the associated HttpServer or HttpsServer._ 

To create a HttpContext instance, you need the createContext(..) method in the HttpServer object. The arguments needed are 'String path' and 'HttpHandler handler'. HttpHandler is an interface with a method `handle​(HttpExchange exchange)`. This method must thus be implemented, in the implemented handle method you tell Java what to do when the url is called, and because HttpExchange is the argument the handle method can work with the request and create a proper response. 

This means that for every valid url on your site you need to generate a HttpContext instance that contains both the url that it applies to and the actions to be taken, described in the lambda expression.

HttpContext objects are stored inside the HttpServer object. To remove them, you need to call the removeContext method on HttpServer.

## Code by ChatGPT

ChatGPT offered to write basic code to make the Http Server work. It took a while before I completely understood it, having lambda's as arguments always requires some more thinking. Anyhow, I'll show the code and will discuss it.

### The Router class

```
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpServer;

import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.util.*;
import java.util.regex.*;

public class Router {

    private final HttpServer server;
    private final List<Route> routes = new ArrayList<>();

    public Router(int port) throws IOException {
        server = HttpServer.create(new InetSocketAddress(port), 0);
        server.createContext("/", this::dispatch);
    }

    public void start() {
        server.start();
        System.out.println("Router started on port " + server.getAddress().getPort());
    }

    public void get(String path, RouteHandler handler) {
        addRoute("GET", path, handler);
    }

    public void post(String path, RouteHandler handler) {
        addRoute("POST", path, handler);
    }

    private void addRoute(String method, String path, RouteHandler handler) {
        routes.add(new Route(method, compilePath(path), handler));
    }

    private Pattern compilePath(String path) {
        // Convert "/users/{id}" into a regex "/users/(?<id>[^/]+)"
        String regex = path.replaceAll("\\{([^/}]+)\\}", "(?<$1>[^/]+)");
        return Pattern.compile("^" + regex + "$");
    }

    private void dispatch(HttpExchange exchange) throws IOException {
        String method = exchange.getRequestMethod();
        String rawPath = exchange.getRequestURI().getPath();

        for (Route route : routes) {
            if (route.method.equals(method)) {
                Matcher m = route.path.matcher(rawPath);
                if (m.matches()) {
                    Map<String, String> pathParams = new HashMap<>();
                    for (String name : route.path.namedGroups()) {
                        pathParams.put(name, m.group(name));
                    }

                    route.handler.handle(exchange, pathParams);
                    return;
                }
            }
        }

        byte[] msg = "Not Found".getBytes();
        exchange.sendResponseHeaders(404, msg.length);
        exchange.getResponseBody().write(msg);
        exchange.close();
    }

    private static class Route {
        String method;
        Pattern path;
        RouteHandler handler;

        Route(String method, Pattern path, RouteHandler handler) {
            this.method = method;
            this.path = path;
            this.handler = handler;
        }
    }
}
```

### Imports

Let's start at the top with the imports. Only classes HttpServer and HttpRequest are imported, as HttpContext is created with a method from HttpServer. So these two are the basic imports. Furthermore there is `import java.net.InetSocketAddress;`, which is a class needed as argument upon creating the HttpServer object. It contains both IP address and port number.

### HttpContext

It would be logical to provide a context for every endpoint you want to serve but ChatGPT proposes to only add one HttpContext using this line:

```
server.createContext("/", this::dispatch);
```

I asked about the logic and ChatGPT explained that this is a smart solution. As I understand it every request is handled by this context, more specifically by the `private void dispatch(HttpExchange exchange) throws IOException` method. This method has a signature identical to the method signature of the SAM (Single Abstract Method) in functional interface HttpHandler, which is the required argument:

```
@FunctionalInterface
public interface HttpHandler {
    void handle(HttpExchange exchange) throws IOException;
}
```

I was not aware of it, I thought that the method used as argument should either be a lambda with the same arguments and return value as 'handle', or that you could provide an anonymous class based on HttpHandler. But I learned that you can provide a method that has the same signature (or actually, not everyting in the signature has to be completely identical, there is some refinement in the rules).

The gist of this all is that the dispatch method does the actual routing. For clarity, here is the method:

```
    private void dispatch(HttpExchange exchange) throws IOException {
        String method = exchange.getRequestMethod();
        String rawPath = exchange.getRequestURI().getPath();

        for (Route route : routes) {
            if (route.method.equals(method)) {
                Matcher m = route.path.matcher(rawPath);
                if (m.matches()) {
                    Map<String, String> pathParams = new HashMap<>();
                    for (String name : route.path.namedGroups()) {
                        pathParams.put(name, m.group(name));
                    }

                    route.handler.handle(exchange, pathParams);
                    return;
                }
            }
        }
```

So now, if any request comes in, it goes through this method, which compares its URI and method (GET or POST) with a list of Route objects. 

### The Route object list

The class contains this nested static class:

```
    private static class Route {
        String method;
        Pattern path;
        RouteHandler handler;

        Route(String method, Pattern path, RouteHandler handler) {
            this.method = method;
            this.path = path;
            this.handler = handler;
        }
    }
```

It is just a container that can be compared to incoming requests (using method and path fields). If there is a match, the RouteHandler handler method is called.

### RouteHandler

To make it work, a definition of RouteHandler is required:

```
@FunctionalInterface
public interface RouteHandler {
    void handle(HttpExchange exchange, Map<String, String> pathParams) throws IOException;
}
```

It is a functional interface with a SAM (Single Abstract Method) .handle(..). An implementation of this method is being called in the dispatch method once a match has been found between an incoming request and a Route object. The line in dispatch responsible for the call is this line:

```
route.handler.handle(exchange, pathParams);
```

### How the Route objects are created

The Route objects define how specific requests are to be handled. The creation of Route objects happens in the main class, after the creation or the Router instance. This is the code for Main:

```
public class Main {
    public static void main(String[] args) throws Exception {

        Router router = new Router(80);   // your container maps this externally

        router.get("/hello", (exchange, path) -> {
            String response = "Hello!";
            exchange.sendResponseHeaders(200, response.length());
            exchange.getResponseBody().write(response.getBytes());
            exchange.close();
        });

        // Example: /users/123
        router.get("/users/{id}", (exchange, path) -> {
            String userId = path.get("id");
            String json = "{ \"user\": \"" + userId + "\" }";

            exchange.getResponseHeaders().add("Content-Type", "application/json");
            exchange.sendResponseHeaders(200, json.length());
            exchange.getResponseBody().write(json.getBytes());
            exchange.close();
        });

        // POST example
        router.post("/users", (exchange, path) -> {
            String jsonBody = RouterUtils.body(exchange);
            // do something with it

            String response = "Received POST: " + jsonBody;
            exchange.sendResponseHeaders(200, response.length());
            exchange.getResponseBody().write(response.getBytes());
            exchange.close();
        });

        router.start();
    }
}
```

There are two methods from Router involved, namely 'get' for GET requests and 'post' for POST requests. Calling them creates a Route object containing method, path and RouteHandler method. So right now, the Main class provides the code that defines what would be in a Controller class if this was a Spring project.

### Helper methods

ChatGPT also provided two helper methods to assist in the process, one to extract query parameters and one to read the body of a POST request:

```
public static Map<String, String> queryParams(HttpExchange exchange) {
    Map<String, String> params = new HashMap<>();
    String query = exchange.getRequestURI().getQuery();
    if (query != null) {
        for (String pair : query.split("&")) {
            String[] parts = pair.split("=");
            if (parts.length == 2) {
                params.put(parts[0], parts[1]);
            }
        }
    }
    return params;
}
```

```
public static String body(HttpExchange exchange) throws IOException {
    return new String(exchange.getRequestBody().readAllBytes());
}
```

This is what you get when not working in Spring. I'm gonna use this code as inspiration for the project and by studying it I learned more about this fine little httpserver package.


