# GET, PUT, POST, DELETE and RestClient

Module 5 of the Spring Boot chapter is named 'RESTFul Application with Spring Boot' and mainly deals with the @RestController class. It can be divided in two different parts, corresponding to a different role of your application:

## Two themes - overview

### _Your application is a server_

This requires knowledge of the following annotations: 

- @GetMapping 
- @PutMapping
- @PostMapping  // most difficult because of Location header
- @DeleteMapping
- @ResponseStatus
- @RequestBody 

The ones we assume to be known are @RequestMethod, @PathVariable, @RequestHeader and @RequestParam (see previous blog post).

We deal with response status (only for successful operations) and with the Location header in case of @PostMapping. We appreciate the Message Convertors that bring so much convenience, and we learn how to work with `ServletUriComponentsBuilder` to build the Location URI we want to have in our response to a POST request. We revisit ResponseEntity, which was already in the previous blog post.

### _Your application is a client_

Here the `RestTemplate` object is the star of the show. It has been sort of surpassed by WebClient but is not deprecated, it will only not evolve any further. RestTemplate supports all HTTP methods. The great thing is the conversion of types, everything that it sends or receives is automatically converted from web-like format to Java format.

## What is REST

Before going to the methods and annotations, a little overview. REST is:

- REpresentional State Transfer, term coined by Roy Fielding
- HTTP as application protocol, not just as transport
- Emphasizes scalability

### Rest principles:

- Expose resource through URIs
    - Model nouns, not verbs
- Resources support limited set of operations
    - GET, PUT, POST, DELETE in case of HTTP
    - All have well-defined semantics
- To know the intention of an URI it needs to be combined with http method (GET, PUT, POST, DELETE). The URI itself is just a 'resource'.
- Clients can request particular representation
    - Resources can support multiple representations (JSON, XML, ..)
- Representations can link to other resources
    - allows for extensions and discovery, like with web sites.
- Hypermedia As The Engine Of APplication State (HATEOAS)
    - RESTful response contain the links you need - just like html pages do
- Stateless architecture
    - No HttpSession usage
    - GETs can be cached on URL
    - Recommends clients to keep track of the state
    - Part of what makes it scalable
    - Looser coupling between client and server
- HTTP headers and status codes communicate result to clients
    - All well-defined in HTTP Specification

## GET, PUT, POST, DELETE (server)

### @GetMapping

`@GetMapping("/store/orders")` is a shortcut for `@RequestMapping(path="/store/orders", method=RequestMethod.GET)`. You can use @RequestMapping for all http methods, and you have more options to customize the response. I learned from ChatGPT that in earlier Spring versions everything was done with @RequestMapping and that @GetMapping, @PutMapping etc were introduced more recently.

@RequestMethod also lets you select the methods PATCH, HEAD, OPTIONS and TRACE. For these methods no @[]Mapping annotation exist.

The Message Convertors, autoconfigured by Spring, do all the conversion, not only for what you send but also for what you receive as response. Spring MVC resolves the desired content type based on Accept reuest header.

### @PutMapping

PUT is used to update a record. If a client sends a PUT request and everything goes fine, we need to return status code 204, or HttpStatus.NO_CONTENT. This basically says that everything went fine and that the body of the response is empty. If the body is empty, this means that our @PutMapping method has void return type. This is a sample using @ResponseStatus to send 204:

```
@PutMapping("/store/orders/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT) // 204
public void updateOrder(..){

	// update order
}
```

In case of an updated or new record, the client will send body content. This can be accessed using the @RequestBody annotation. Note that Message Converter immediately converts this body content to the domain type we work with. This decouples the web-environment (JSON, XML) from our application (domain objects) and makes testing much easier. Example:

```
@PutMapping("/store/orders/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT) // 204
public void updateOrder(@RequestBody Order updatedOrder, @PathVariable long id){

	orderManager.updateOrder(id, updatedOrder);
}
```

### @PostMapping

If a client sends a new record with POST, we return status '201 Created' upon success. The URI (url) where the new record is sent as value for the 'Location' header. No body content is sent (Content-Length: 0). The creation of this location URI requires some skill. 

The method we write using @PostMapping is very similar to that of @PutMapping, but now we need to generate the Location URI and get it into the response. To do so you can use the ResponseEntity<Void> type object. ResponseEntity allows us to create new headers in the response. Therefore we can also omit the @ResponseStatus annotation, as ResponseEntity can set the response status.

There are two classes that help with building the URI, namely `UriComponentsBuilder` and `ServletUriComponentsBuilder`. The latter is a subclass of the former and has better functionality, as it provides access to the URL that invoked the current controller method.

The video tutorial shows this example that does everything we need: the body of the POST request is stored in our system, a Location header with the right URI is created, and the response contains status code '201 Created' plus a Location header with the correct URI. The Void in `ResponseEntity<Void>` is there because the body of the response is empty. 

```
@PostMapping("/store/orders/{id}/items")
public ResponseEntity<Void> createItem(@PathVariable long id, @RequestBody Item newItem){

	// add item
	orderService.findOrderById(Id).addItem(newItem);

	// Build the location URI of the newly added item
	URI location = ServletUriComponentsBuilder
		.fromCurrentRequestUri()
		.path("/{itemId}")
		.buildAndExpand(newItem.getId())
		.toUri();

	// Explicitly create a '201 Created' response with Location header set
	return ResponseEntity.created(location).build();
}
```

### @DeleteMapping

If a client sends a DELETE request, there is no body in it. The response will have '204 No Content' status and no body either. This is relatively simple. No ResponseEntity or ServletUriComponentsBuilder is needed.

```
@DeleteMapping("/store/orders/{orderId}/items/{itemId}")
@ResponseStatus(HttpStatus.NO_CONTENT) // 204
public void deleteItem(@PathVariable long orderId, @PathVariable String itemId) {

	orderService.findOrderById(orderId).deleteItem(itemId);
}
```

## You are the client











