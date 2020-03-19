# Node HTTP server

Node is most often used to create HTTP servers for the web. It has some nice built-in tools that help us do this.

## Workshop prep

1. Clone this repo
1. Open `workshop/server.js` in your editor
1. Run `node workshop/server.js` in your terminal whenever you want to execute your code

Follow along with each example in your own editor.

## Creating a server

The built-in `http` module has a `createServer` method.

```js
const http = require("http");

const server = http.createServer();
```

## Handling requests

Our server currently does nothing. We need to pass a "handler function" to `createServer`. This function will be run whenever the server receives a request. This is similar to `addEventListener` in the browser.

The handler will be passed two arguments: an object representing the incoming request, and an object representing the response that will eventually be sent.

```js
const http = require("http");

const server = http.createServer((request, response) => {
  response.end("hello");
});
```

We can use the `end` method on the response object to tell Node to send the response. Whatever we pass here will be sent as the response body.

## Starting the server

Our Node program has a functioning server, but that server isn't currently listening for requests. We need to tell it to do so, and what port it should listen on:

```js
const http = require("http");
const PORT = 3000;

const server = http.createServer((request, response) => {
  response.end("hello");
});

server.listen(PORT, () => console.log(`Listening on http://localhost:${PORT}`));
```

We use the `listen` method of the server object. This takes the port number to listen on, and an optional callback to run when it starts listening. This callback is a good place to log something so you know the server has started.

Now we can run the program in our terminal with `node server.js`. The server will start and you should see "Server listening on http://localhost:3000" logged.

## Sending requests

We can now send HTTP requests to our server and we should see the "hello" response. You can open http://localhost:3000 in your browser to send a `GET` request and see the response. It's helpful to open the network tab of devtools so you can see all the details of the request and response.

The server currently returns "hello" as plaintext no matter what request we send it (try visiting random endpoints like http://localhost:3000/shenanigans). Lets make it more interesting.

## The response

HTTP responses need a few different things:

1. A status code (e.g. `200` for success or `404` for not found)
1. Headers to provide info about the response
1. A body (the response data itself)

### Status code

We're currently only providing the body. Node will set the status code to `200` by default, but it's best to be explicit.

```js
const server = http.createServer((request, response) => {
  response.statusCode = 200;
  response.end("Hello");
});
```

### Headers

We should set headers describing our response. For example here we're sending the body as regular text, so we should tell the browser that using the `content-type` header.

```js
const server = http.createServer((request, response) => {
  response.statusCode = 200;
  response.setHeader(("content-type": "text/plain"));
  response.end("Hello");
});
```

### JSON body

We aren't limited to a plaintext response. Lets send some JSON instead.

```js
const server = http.createServer((request, response) => {
  response.statusCode = 200;
  response.setHeader(("content-type": "application/json"));
  response.end(JSON.stringify({ message: "Hello" }));
});
```

It's important to note that our response has to be a string, which is why we stringify our object.

### HTML body

Browsers don't handle JSON that well. Web pages are made of HTML, so let's change our response to that.

```js
const server = http.createServer((request, response) => {
  response.statusCode = 200;
  response.setHeader(("content-type": "text/html"));
  response.end("<h1>Hello</h1>");
});
```

### Simplifying headers

If we wanted to set more headers we'd end up with a lot of `setHeader` calls. Node has a method for setting the status code _and_ all the headers at once: `response.writeHead`.

```js
const server = http.createServer((request, response) => {
  response.writeHead(200, { "content-type": "text/html" });
  response.end("<h1>Hello</h1>");
});
```

We provide the status code as the first argument and an object of headers as the second.

## The request

So far we've only looked at the response argument of our handler function. Let's log a few properties of the request object.

```js
const server = http.createServer((request, response) => {
  console.log(request.method, request.url);
  response.writeHead(200, { "content-type": "text/html" });
  response.end("<h1>Hello</h1>");
});
```

Now when you refresh http://localhost:3000 in your browser you should see `GET /` logged in your terminal. Now visit http://localhost:3000/goodbye in your browser. You should see `GET /goodbye` logged in your terminal. Now run `curl -X POST localhost:3000` in a new terminal tab/window (or use Postman to send a POST request if you prefer). You should see `POST /` logged by your server.

We can use the method and URL of the request to determine what response to send.

## Routing

Lets create another page for our site that displays a different message at `/goodbye`.

```js
const server = http.createServer((request, response) => {
  const url = request.url;
  if (url === "/") {
    response.writeHead(200, { "content-type": "text/html" });
    response.end("<h1>Hello</h1>");
  } else if (url === "/goodbye") {
    response.writeHead(200, { "content-type": "text/html" });
    response.end("<h1>Goodbye</h1>");
  }
});
```

Visit http://localhost:3000/goodbye in your browser and you should see the "Goodbye" title.

### Missing resources

Now that we have routing for different pages it's possible to send requests for resources that don't exist. For example visit http://localhost:3000/uhoh in your browser. The request will hang as the browser never receives a response.

We should add a case to our `if` statement to send a response when the URL doesn't match.

```js
const server = http.createServer((request, response) => {
  const url = request.url;
  if (url === "/") {
    response.writeHead(200, { "content-type": "text/html" });
    response.end("<h1>Hello</h1>");
  } else if (url === "/goodbye") {
    response.writeHead(200, { "content-type": "text/html" });
    response.end("<h1>Goodbye</h1>");
  } else {
    response.writeHead(404, { "content-type": "text/html" });
    response.end("<h1>Not found</h1>");
  }
});
```

Now our server will send a `404` response for any request it doesn't recognise, and the user will see a "Not found" message.

## Request bodies

Todo: `POST` requests, incoming body stream

### Dynamic responses

Using template literals to include dynamic data in HTML response