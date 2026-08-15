
## HTTP headers

### What are HTTP Headers?

HTTP Headers are key-value pairs included in an HTTP Request or Response. They provide metadata about the request or response, such as authentication details, content type, caching policies, and more.

### Types of HTTP Headers

- **Request Headers:** Sent by the client to provide additional information about the request. 
    Example: `Authorization: Bearer {access_token}`
- **Response Headers:** Sent by the server to provide additional information about the response.  
    Example: `Content-Type: application/json`

### Common HTTP Headers

HTTP Headers are key-value pairs included in an HTTP Request or Response. They provide metadata about the request or response, such as authentication details, content type, caching policies, and more.

![[Pasted image 20260701194419.png]]

### Why Use HTTP Headers?

- **Authentication:** Headers like Authorization ensure secure access to resources.
- **Content Negotiation:** Headers like Content-Type and Accept ensure the client and server agree on the data format.
- **Performance Optimization:** Headers like Cache-Control improve performance by enabling caching.
- **Debugging and Analytics:** Headers like User-Agent and Referer provide insights into client behavior.

### When to Use HTTP Headers?

- Always include authentication headers for secure API calls.
- Use Content-Type and Accept headers when sending or receiving structured data (e.g., JSON).
- Use caching headers (Cache-Control) to optimize performance for frequently accessed resources.

### Benefits of HTTP Headers

- **Security:** Authentication headers protect sensitive resources.
- **Flexibility:** Headers allow customization of requests and responses.
- **Efficiency:** Caching headers reduce server load and improve response times.

### Consequences of Not Using HTTP Headers

- **Security Risks:** Without authentication headers, unauthorized users may access sensitive resources.
- **Data Mismatch:** Omitting Content-Type or Accept headers can lead to errors in data processing.
- **Performance Issues:** Without caching headers, servers may become overloaded with redundant requests.



## HTTP Query Parameters

### What are HTTP Query Parameters?

Query Parameters are key-value pairs appended to the URL of an HTTP request. They are used to pass additional information to the server, such as filters, search criteria, or configuration options.

### Structure of Query Parameters

Query parameters are appended to the URL after a `?` symbol. Multiple parameters are separated by an `&`.

Example: `[https://api.example.com/resources?filter=active&sort=asc](https://api.example.com/resources?filter=active&sort=asc)`

### Why Use Query Parameters?

- **Efficiency:** Retrieve only the data you need, reducing the size of the response.
- **Flexibility:** Customize requests without modifying server-side logic.
- **Standardization:** Query parameters follow a standard format, making them easy to use and understand.

### Consequences of Not Using Query Parameters

- **Overhead:** Including unnecessary parameters can increase response size and complexity.
- **Data Leakage:** Sensitive information in query parameters can be exposed in browser history or logs.


## Query Parameters vs Request Parameters

### Using Query Parameters

Query parameters are used to send data to the server through the URL query string. An example of a URL with a query parameter would look like this: `http://127.0.0.1:5001/api/users?id=123`. In this example, "id" is the query parameter key, and "123" is the query parameter value.

### Request parameters

This is a broader, less formal term that usually refers to **any** data sent along with an HTTP request, regardless of where it lives. That includes:

1. **Query parameters** (in the URL, as above)
2. **Path parameters** — part of the URL path itself, like the `123` in `/users/123`
3. **Body parameters** — data sent in the request body, usually JSON, common in POST/PUT requests
4. **Headers** — things like `Authorization` or `Content-Type` sent in HTTP headers

So "request parameter" is really an umbrella term, and "query parameter" is one specific kind of request parameter — the one that lives in the URL's query string.

### Query Parameters vs. Path Parameters

**Path parameters identify a specific resource; query parameters modify or filter a collection/request.**

Query parameters are ideal for **filtering data or passing optional parameters** to an API endpoint. For example, if you’re building an API endpoint that returns a list of users, you might want to filter the results by age, gender, or location. You can achieve this by using query parameters.

Path parameters are ideal for passing **mandatory parameters** to an API endpoint. For example, if you’re building an API endpoint that returns a user’s information by their ID, you’ll want to pass the ID as a request (path) parameter.