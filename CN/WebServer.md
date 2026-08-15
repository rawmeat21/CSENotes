## What is a Web Server?

“Web Server” can be refer to both software or hardware.

Web Server as a hardware is a computer that stores web server software and a website’s component files (for example, HTML documents, images, CSS stylesheets, and JavaScript files). A web server connects to the Internet and supports physical data interchange with other devices connected to the web.

Web Server as a software includes several parts that control how web users access hosted files. Or you can call it is an _HTTP server_. An HTTP server is software that understands [URLs](https://developer.mozilla.org/en-US/docs/Glossary/URL) (web addresses) and [HTTP](https://developer.mozilla.org/en-US/docs/Glossary/HTTP) (the protocol your browser uses to view webpages). An HTTP server can be accessed through the domain names of the websites it stores, and it delivers the content of these hosted websites to the end user’s device.

## How does a Web Server work?

At the most basic level, whenever a browser needs a file that is hosted on a web server, the browser requests the file via HTTP. When the request reaches the correct (hardware) web server, the (software) _HTTP server_ accepts the request, finds the requested document, and sends it back to the browser, also through HTTP. (If the server doesn’t find the requested document, it returns a [404](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404) response instead.)

**To publish a website, you need either a static or a dynamic web server.

-> A static web server, or stack, consists of a computer (hardware) with an HTTP server (software). We call it “static” because the server sends its hosted files as-is to your browser.

-> A dynamic web server consists of a static web server plus extra software, most commonly an _application server_ and a _database_. We call it “dynamic” because the application server updates the hosted files before sending content to your browser via the HTTP server.

For example, to produce the final webpages you see in the browser, the application server might fill an HTML template with content from a database.


## How Browsers communicate with Servers

Web browsers communicate with [web servers](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_is_a_web_server) using the HyperText Transfer Protocol ([HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)). When you click a link on a web page, submit a form, or run a search, the browser sends an _HTTP Request_ to the server.

This request includes:

- A URL identifying the target server and resource (e.g. an HTML file, a particular data point on the server, or a tool to run).
- A method that defines the required action ( GET, POST, PUT, PATCH,…)
- Additional information can be encoded with the request (for example, HTML form data, URL parameters, or cookies)

Web servers wait for client request messages, process them when they arrive, and reply to the web browser with an HTTP Response message. The response contains an [HTTP Response status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) indicating whether or not the request succeeded (e.g. “`200 OK`" for success, "`404 Not Found`" if the resource cannot be found, "`403 Forbidden`" if the user isn't authorized to see the resource, etc.). The body of a successful response to a `GET` request would contain the requested resource.

When an HTML page is returned it is rendered by the web browser. As part of processing the browser may discover links to other resources (e.g. an HTML page usually references JavaScript and CSS pages), and will send separate HTTP Requests to download these files.

Both static and dynamic websites (discussed in the following sections) use exactly the same communication protocol/patterns.

