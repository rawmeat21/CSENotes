## Main parts of a browser (High-level)

It consists of:

- **User Interface**
- **Browser Engine**
- **Rendering Engine**
- **Networking**
- **JS Engine**
- **UI Backend**
- **Disk API / Data Storage**

![[Pasted image 20260615125741.png]]

### 1. User Interface

This is the part of the browser you actually see, the obvious parts like the address bar, back/forward buttons, bookmarks, tabs, and settings.

### 2. Browser Engine

It the **manager**. Its main job is to connect the UI with the Rendering Engine. That’s why it sits right in the middle of them, helping both of them to interact with each other.

### 3. Rendering Engine

This is the **actual worker,** it is **Rendering** something so. It takes the HTML and CSS and renders them on the screen. The actual website you see and interact with is the result of this engine doing its job.

### 4. Networking

This part handles everything involving communication over the internet. 

It handles DNS lookups to find IPs and manages HTTP/HTTPS requests to fetch the website’s resources.

It also handles things like SSL/TLS handshakes to keep your connection secure and manages “keep-alive” connections so the browser doesn’t have to restart the conversation with the server for every single image or file.

### 5. JS Engine

This part handles the JavaScript. It processes the logic and interactivity like what happens when you click a button or how an animation triggers etc.

### 6. UI Backend

This interacts with the browser’s User Interface. It handles the “under the hood” drawing of basic widgets like windows and combo boxes, often using the operating system’s own UI methods.

### 7. Disk API (Data Storage)

This handles the storage side of things. It manages the cache, cookies, and sessions. It also interacts with the OS for storage purposes and manages how the browser asks for RAM.


## How Browsers Display a website ?

![[Pasted image 20260615130016.png]]

Basically, there will be an HTTP server (like a computer far away) holding your files: HTML, CSS, and JS.

The browser asks _Where is this website?_ It uses DNS to find the IP. Once it has the IP, the browser sends an **HTTP Request**, and the server responds with those documents.

If its an HTML file, its sent to the HTML parser.

**HTML is different. It uses an “Unconventional” parser. It was designed to be “error-tolerant” If you forget to close a `<div>` tag or mess up your syntax, the HTML parser won't crash your site. It tries its best to guess what you meant and keeps going. This is why HTML never really throws errors on your screen.**

As the HTML parser works, it builds the **DOM (Document Object Model)**. At the same time, when the browser sees a `<link>` tag for CSS, it fetches that file and runs it through the **CSS Parser** to create the **CSSOM (CSS Object Model)**.

-> **DOM:** Dom is a tree-like structure of your HTML. Every tag becomes a node. The browser needs this to find elements, change text, and react to your clicks.

-> **CSSOM:** It is a similar tree, but for styles. It maps out which styles belong to which nodes.


Once the browser has the DOM and CSSOM, they go into a **Frame Constructor** to create a Frame tree which is also known as reflow. This kicks off the final three steps:

1. **Layout (Reflow):** This is when the browser calculates the **s**ize and position of every element on the page. It has to consider things like box model, CSS rules, flex/grid layouts, font sizes, etc. This can become very expensive if there are many DOM elements or if elements change size/position frequently. Example triggers are changing `width`, `height`, `padding`, `margin`, `font-size`, or adding/removing elements.
2. **Painting:** The browser fills in the pixels. This is where colors, shadows, and borders finally appear. This is usually less expensive than reflow, because the browser can just redraw the visual pixels without recalculating layout. Example triggers are changing `color`, `background-color`, `visibility` (without affecting layout).
3. **Display:** And after all this the website get rendered on the browser screen and shown to the user.


References:

https://medium.com/@pulkit8129/how-a-browser-works-a-beginner-friendly-guide-to-browser-internals-345de0923c6f
