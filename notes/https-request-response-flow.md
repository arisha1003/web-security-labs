# How an HTTPS Request Travels from Browser to Server and Back

When we type a website such as `google.com` into the browser and press Enter, many processes happen before the webpage appears on our screen. The browser has to find the server, establish a connection, securely communicate with it, send the request, receive the response, and finally render the webpage.

Here is the complete process in simple steps.

## 1. DNS Resolution

The first important step is **DNS resolution**.

Humans use domain names such as:

`google.com`

but computers communicate using IP addresses. Therefore, the browser needs to find the IP address associated with the domain.

The browser may first check its own DNS cache. If the required information is not there, the operating system's DNS cache may be checked.

If the IP address is still not available, the DNS query is sent to a **recursive DNS resolver**. The resolver can use its own cache or query other DNS servers until it obtains the required IP address.

Once the IP address is obtained, the browser knows where it needs to establish the connection.

---

## 2. TCP Three-Way Handshake

For traditional HTTP/1.1 and HTTP/2 connections, the next step is establishing a TCP connection.

TCP uses a **three-way handshake**:

1. Client → Server: `SYN`
2. Server → Client: `SYN-ACK`
3. Client → Server: `ACK`

This establishes a reliable TCP connection between the client and server.

The purpose of the handshake is to synchronize the connection between both sides and prepare TCP for reliable communication.

---

## 3. TLS Handshake

Because we are using **HTTPS**, the connection also needs to be secured using TLS.

TLS stands for **Transport Layer Security**. It is a security protocol that provides:

* Encryption
* Authentication
* Integrity

The TLS handshake allows the client and server to establish the cryptographic parameters and keys that will be used to protect their communication.

After TLS is successfully established, the HTTP data can be sent securely.

So, in a simplified HTTP/1.1 HTTPS connection:

**DNS → TCP handshake → TLS handshake → HTTP communication**

> Note: HTTP/3 works differently because it uses QUIC over UDP rather than TCP.

---

## 4. The HTTP Request

After the secure connection has been established, the browser sends an HTTP request to the server.

A simplified HTTP request can look like this:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 29
Cookie: session=abc123
User-Agent: Mozilla/5.0

username=ali&password=123456
```

The request contains several important parts.

### Request Line

The first line is the **request line**:

```http
POST /login HTTP/1.1
```

It contains three main parts:

* `POST` → HTTP method
* `/login` → requested path
* `HTTP/1.1` → HTTP version

The HTTP method tells the server what kind of operation the client is requesting.

For example:

* `GET` → commonly used to retrieve data
* `POST` → commonly used to submit data
* `PUT` → commonly used to update or replace data
* `DELETE` → commonly used to delete data

A POST request commonly sends submitted data in the request body.

---

## 5. HTTP Request Headers

After the request line, the browser sends **request headers**.

Headers contain additional information about the request.

Examples include:

```text
Host
Content-Length
Content-Type
Cookie
User-Agent
```

For example, the `Cookie` header can contain information that helps the server identify an existing session.

The `Content-Type` header tells the server what type of data is being sent.

The `User-Agent` header provides information about the client software making the request.

---

## 6. HTTP Request Body

Some HTTP requests also contain a **request body**.

For example, when submitting a login form using POST, the body could contain:

```text
username=ali&password=123456
```

The body contains the actual data being submitted to the server.

However, not every HTTP request has a body. For example, a typical GET request does not normally need one.

---

# 7. The Request Travels Through the Web Infrastructure

The request now travels through the network toward the destination.

An important thing to understand is that **not every web application has the same architecture**.

Depending on the application's requirements, the request may pass through technologies such as:

* CDN
* WAF
* Reverse proxy
* Load balancer
* Web/application server
* Database

A small application may have a much simpler architecture, while a large application may have many layers.

These technologies are not required in exactly the same order for every website.

---

## 8. CDN

A **Content Delivery Network (CDN)** can cache and deliver certain types of content from servers located closer to users.

For example, static resources such as:

* Images
* CSS
* JavaScript
* Videos
* Other cacheable files

may be served from a nearby CDN edge server instead of going all the way to the origin server.

This can reduce latency and reduce the amount of traffic reaching the origin infrastructure.

Some CDN providers also provide additional services such as load balancing and security features.

---

## 9. WAF

A **Web Application Firewall (WAF)** can inspect HTTP/HTTPS traffic and apply security rules.

For example, it may look for request patterns associated with common web attacks and decide whether a request should be allowed, blocked, or challenged.

The exact placement of a WAF depends on the application's architecture. It may be provided as part of a CDN or deployed separately.

---

## 10. Reverse Proxy

A **reverse proxy** sits in front of backend servers and receives requests on their behalf.

Technologies such as **Nginx** and **Apache** can be configured to work as reverse proxies.

A reverse proxy can perform tasks such as:

* Forwarding requests to backend servers
* TLS termination
* Load balancing
* Routing
* Serving static content
* Adding or modifying headers

For example:

```text
Client
   ↓
Reverse Proxy
   ↓
Application Server
```

The reverse proxy helps separate the public-facing part of the infrastructure from the backend application.

---

## 11. Application/Web Server

Eventually, the request reaches the application's backend.

The exact technology depends on the application.

For example, an application might use:

* PHP
* Python with Flask or Django
* Node.js
* Java
* .NET

The application processes the request and decides what response should be returned.

For example, for a login request, the application may take the submitted username and password and perform the required authentication logic.

---

## 12. Database

The application may need information from a database to complete the request.

Databases can store information such as:

* User accounts
* Names
* Email addresses
* Application data
* Product information
* Session-related information

The application communicates with the database using the appropriate database interface and query language.

For example, during a login process, the application may retrieve the account information associated with the submitted username and then perform the required authentication checks.

The database is normally not directly exposed to the public internet. It is usually accessed by backend application components.

---

# 13. The Server Creates the HTTP Response

After the backend has finished processing the request, the server sends an HTTP response back to the client.

A simplified response might look like:

```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 5234
Set-Cookie: session=abc123

<html>
    ...
</html>
```

The first line is the **status line**.

```http
HTTP/1.1 200 OK
```

It contains:

* HTTP version
* Status code
* Reason phrase

Some common status codes are:

* `100` → Informational
* `200` → OK
* `301` → Moved Permanently
* `302` → Found / Redirect
* `404` → Not Found
* `500` → Internal Server Error

Status codes are divided into different categories:

* `1xx` → Informational
* `2xx` → Successful
* `3xx` → Redirection
* `4xx` → Client-side errors
* `5xx` → Server-side errors

---

# 14. Response Headers

After the status line, the server sends **response headers**.

These provide additional information about the response.

Examples include:

```text
Content-Type
Content-Length
Set-Cookie
Cache-Control
Location
Content-Security-Policy
```

For example, `Set-Cookie` can instruct the browser to store a cookie.

Security-related headers can also tell the browser how certain resources and behaviors should be handled.

---

# 15. Response Body

The response may also contain a **response body**.

For a webpage, this might contain HTML:

```html
<html>
    <body>
        <h1>Hello</h1>
    </body>
</html>
```

However, the response body is not always HTML.

It could also contain:

* JSON
* JavaScript
* CSS
* Images
* Videos
* Other application data

The type of content is generally indicated by the `Content-Type` response header.

---

# 16. The Response Travels Back to the Browser

The response travels back through the established network connection.

The browser receives the HTTP response and processes:

* Status code
* Response headers
* Response body
* Cookies
* Security-related information
* Other metadata

The browser then determines what it needs to display and what additional resources need to be requested.

For example, an HTML document may reference:

```text
style.css
script.js
logo.png
```

The browser may then make additional HTTP requests for those resources.

---

# 17. Browser Parsing and Rendering

Finally, the browser processes the received content.

For HTML, the browser parses the document and creates the **DOM (Document Object Model)**.

CSS is processed to determine how elements should be styled, and JavaScript can modify the DOM and interact with the page.

The browser then performs the necessary rendering work and displays the webpage on the screen.

This is why the webpage we see is the final result of many processes that happened behind the scenes.

---

# Complete Flow

A simplified HTTPS request flow can be represented as:

```text
User enters URL
       ↓
DNS Resolution
       ↓
IP Address Obtained
       ↓
TCP Three-Way Handshake
       ↓
TLS Handshake
       ↓
HTTP Request
       ↓
CDN / WAF / Reverse Proxy
       ↓
Application Server
       ↓
Database (if required)
       ↓
Application Processes Request
       ↓
HTTP Response
       ↓
Reverse Proxy / CDN / Network
       ↓
Browser Receives Response
       ↓
HTML/CSS/JavaScript Processing
       ↓
DOM + Rendering
       ↓
Webpage Appears
```

## Important Note About Web Architecture

The flow above is a simplified model.

A real-world web application does not necessarily use all of these technologies.

For example, one application might have:

```text
Browser → Server → Database
```

while a large application might have:

```text
Browser
   ↓
CDN
   ↓
WAF
   ↓
Load Balancer
   ↓
Reverse Proxy
   ↓
Application Servers
   ↓
Database / Cache / Other Services
```

The architecture depends on the application's requirements, scale, performance needs, security requirements, and design.

## What I Learned

An HTTPS request is not simply:

**Browser → Server → Response**

There are multiple layers involved.

First, DNS helps the browser find the server's IP address. Then TCP establishes a reliable connection, and TLS secures the communication. After that, HTTP is used to send the actual request.

The request may pass through different infrastructure components such as CDNs, WAFs, reverse proxies, and load balancers before reaching the application. The application may then communicate with a database to process the request.

Finally, the server generates an HTTP response, which travels back to the browser. The browser processes the response, loads additional resources when necessary, builds the DOM and other rendering structures, and finally displays the webpage to the user.

Understanding this complete flow is important for cybersecurity because many security concepts—such as DNS security, TLS, cookies, HTTP headers, WAFs, reverse proxies, authentication, SQL injection, and web-server security—exist at different points in this communication process.

