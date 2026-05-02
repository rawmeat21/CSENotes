### First — what is HTTP?

When your Python program wants data from Codeforces or LeetCode, it can't just read a variable or call a function in their codebase. Their data lives on a server — a computer somewhere that's running software specifically designed to listen for incoming connections and respond to them.

The protocol those servers speak is **HTTP** — HyperText Transfer Protocol. It's the foundational communication protocol of the web. Every time you open a browser and visit a URL, your browser is making HTTP requests. Your Python code does the exact same thing — it just does it programmatically instead of through a GUI.

HTTP is a **request-response protocol**. That means:

1. Your program (the **client**) sends a **request** to a server
2. The server processes it and sends back a **response**
3. The connection closes (in HTTP/1.1 this is technically negotiable, but conceptually each request-response is a unit)

There's no persistent state between requests by default. Every request is independent. This is what "stateless" means in the context of HTTP — the server doesn't inherently remember you from one request to the next. That's why cookies exist (we'll get to that).

---

### HTTP Methods

Every HTTP request specifies a **method** — this is a verb that tells the server what kind of operation you're requesting. There are several methods, but you only use two in your code:

#### GET

A GET request means: _"give me this resource."_ You're asking the server to return some data at a given URL. You are not sending any input data — just the URL itself, and optionally some query parameters tacked onto the URL.

GET requests are:

- **Idempotent** — making the same GET request 10 times should give you the same result each time, and shouldn't change anything on the server
- **Safe** — they are not supposed to cause side effects on the server

In your code, every Codeforces and AtCoder request is a GET:

python

```python
# from _req, line 87
r = requests.get(url, headers=headers, timeout=12)
```

When you call `cf_fetch_problems()`, it calls `_req` with the default method `"GET"`, which hits:

```
https://codeforces.com/api/user.status?handle=lambdadelta&from=1&count=10000
https://codeforces.com/api/problemset.problems
https://kenkoooo.com/atcoder/atcoder-api/v3/user/submissions?user=rawmeat21&from_second=0
```

All of these are just "give me data" requests — no input beyond the URL.

#### POST

A POST request means: _"here is some data — process it and give me a response."_ You're sending a **body** along with the request — a chunk of data that the server reads as input.

POST requests are:

- **Not idempotent** by convention (though in practice your LeetCode requests are read-only)
- Used when you need to send structured input to the server

In your code, every LeetCode request is a POST:

python

```python
# from _req, line 85
r = requests.post(url, headers=headers, json=json_data, timeout=15)
```

LeetCode uses GraphQL, which always uses POST because you're sending a query string as the request body. We'll go deep on GraphQL in Topic 3, but the key point here is: you're sending data _to_ `https://leetcode.com/graphql`, and it processes your query and returns results.

---

### The `requests` library

Python has a built-in module called `urllib` that can make HTTP requests, but it's verbose and low-level. The `requests` library is a third-party package that wraps it with a clean, readable API. It's one of the most downloaded Python packages ever written.

You install it with:

```
pip install requests
```

Your code imports it at line 15:

python

```python
import requests
```

#### `requests.get()`

python

```python
response = requests.get(url)
response = requests.get(url, headers=headers)
response = requests.get(url, headers=headers, timeout=12)
response = requests.get(url, params={"key": "value"})
```

`requests.get()` takes:

- `url` — the full URL string
- `headers` — a dict of HTTP headers to send (optional)
- `timeout` — how many seconds to wait before giving up (optional but important)
- `params` — a dict of query parameters that get appended to the URL automatically (optional). `params={"handle": "user", "count": 100}` becomes `?handle=user&count=100` in the URL

It returns a **Response object**.

#### `requests.post()`

python

```python
response = requests.post(url, json=data)
response = requests.post(url, headers=headers, json=data, timeout=15)
response = requests.post(url, data="raw string body")
```

`requests.post()` takes everything `requests.get()` takes, plus:

- `json` — a Python dict/list that gets serialized to a JSON string and sent as the body. `requests` also automatically sets the `Content-Type: application/json` header when you use this parameter
- `data` — a raw body, either a string or bytes. Used when you're not sending JSON (not used in your code)

The `json=` parameter is important — if you wrote `data=json_data` instead, `requests` would not serialize it and would not set the content-type header. `json=json_data` handles both automatically.

---

### The Response Object

Both `requests.get()` and `requests.post()` return a `Response` object. This object contains everything the server sent back. The key attributes and methods:

#### `response.status_code`

An integer. The HTTP status code the server returned.

python

```python
r = requests.get("https://codeforces.com/api/problemset.problems")
print(r.status_code)  # 200
```

#### `response.text`

The response body as a Python string. Raw text, not parsed.

#### `response.content`

The response body as raw bytes. Used when dealing with binary data (images, files). Not used in your code.

#### `response.json()`

Parses the response body as JSON and returns a Python dict or list. This is what your `_req` function uses:

python

```python
return r.json()  # line 89
```

Internally, `r.json()` just calls `json.loads(r.text)`. It raises a `json.JSONDecodeError` if the response body isn't valid JSON. In practice, if a server returns an error page in HTML (which sometimes happens), `r.json()` will throw — which is why the `except Exception` catch-all in `_req` exists.

#### `response.headers`

A dict-like object of the response headers the server sent back. Useful for things like checking `Content-Type` or reading rate-limit headers. Not used directly in your code but worth knowing.

#### `response.url`

The final URL after any redirects. If the server redirected you from A to B, `response.url` is B.

#### `response.raise_for_status()`

This is a method, not an attribute. It checks `response.status_code` and raises an exception if the status indicates failure.

python

```python
r.raise_for_status()  # line 88
```

Specifically:

- 4xx status codes → raises `requests.exceptions.HTTPError` (client error — something wrong with your request)
- 5xx status codes → raises `requests.exceptions.HTTPError` (server error — something wrong on their end)
- 2xx status codes → does nothing, returns `None`

Without this call, `requests` gives you the response regardless of status code. A 404 or 500 response is not automatically an error from `requests`'s perspective — you have to check yourself. `raise_for_status()` automates that check.

When the exception is raised, the exception object has a `.response` attribute, which is the original Response object. Your code uses this:

python

```python
# line 90-91
except requests.exceptions.HTTPError as e:
    sc = e.response.status_code
```

So even when the request "fails," you can still read the status code to decide _how_ to fail.

---

### Status Codes

Status codes are 3-digit integers grouped by their first digit:

**1xx — Informational.** Rarely seen in practice. Means "I got your request, still processing." Not relevant to your code.

**2xx — Success.**

- `200 OK` — standard success. Request worked, here's the data.
- `201 Created` — something was created (relevant for POST requests that write data, not in your code)
- `204 No Content` — success but no body returned

**3xx — Redirection.**

- `301 Moved Permanently` — the resource is now at a different URL
- `302 Found` — temporary redirect `requests` follows redirects automatically by default, so you won't usually see these in your response object

**4xx — Client Errors.** Something is wrong with _your_ request.

- `400 Bad Request` — malformed request, invalid parameters
- `401 Unauthorized` — you need to authenticate (no credentials provided)
- `403 Forbidden` — server understood your request but refuses to fulfill it (credentials were provided but insufficient, or IP blocked)
- `404 Not Found` — the URL doesn't exist
- `429 Too Many Requests` — you've exceeded the server's rate limit

**5xx — Server Errors.** Something is wrong on _their_ end.

- `500 Internal Server Error` — unhandled exception on the server
- `502 Bad Gateway` — intermediate server (like a proxy or load balancer) got an invalid response from the upstream server
- `503 Service Unavailable` — server is overloaded or down for maintenance
- `504 Gateway Timeout` — the upstream server didn't respond in time

Your code handles this split explicitly:

python

```python
# lines 92-95
if sc in (429, 500, 502, 503, 504):
    time.sleep(2 ** attempt)   # temporary — retry
else:
    return None                # permanent — give up
```

The logic is: 429 and 5xx errors are _transient_ — the server was temporarily overloaded or unavailable. Retrying after a wait makes sense. But 403 or 404 won't fix themselves with a retry, so returning `None` immediately is correct.

---

### Headers

An HTTP request (and response) consists of two parts: **headers** and a **body**.

Headers are metadata — key-value pairs that describe the request. They're sent before the body. The server reads them to understand _how_ to interpret the request.

Common request headers:

#### `Content-Type`

Tells the server the format of the request body. When your code sends a LeetCode GraphQL request:

python

```python
hdrs = {
    "content-type": "application/json",  # line 190
    ...
}
```

This tells LC's server: "the body I'm sending is JSON." Without this, the server might not parse the body correctly.

When you use `requests.post(json=data)`, `requests` sets `Content-Type: application/json` automatically. Your code sets it _manually_ in `hdrs` because it passes `headers=hdrs` explicitly — both paths result in the same header being sent.

#### `User-Agent`

Identifies the client making the request. Browsers send something like `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 ...`. Your AtCoder requests send:

python

```python
hdrs = {"User-Agent": "Mozilla/5.0"}  # line 144
```

Kenkoooo's AtCoder API checks the User-Agent and may block or throttle requests that don't look like a browser. Sending a browser-like value avoids that. This is extremely common when writing scrapers or API clients.

#### `Cookie`

Sends cookies to the server (explained in detail below).

#### `Referer`

Tells the server what URL you're "coming from." LeetCode's solve checker uses it:

python

```python
"referer": "https://leetcode.com",  # line 328
```

LC's GraphQL endpoint checks the referer as a basic anti-automation measure. Sending it makes your request look like it originated from the LC website.

#### `x-csrftoken`

CSRF (Cross-Site Request Forgery) is a class of attack. Web frameworks issue a CSRF token that must be included in POST requests to prove the request came from the legitimate client. LC requires this header for its GraphQL endpoint:

python

```python
"x-csrftoken": "dummy",  # line 329
```

Interestingly, `"dummy"` works here — LC validates the presence of the header more than the actual value in this context. But the header must be there.

---

### Cookies

HTTP is stateless — each request is independent. But websites need to know who you are across multiple requests. Cookies solve this.

Here's how the cookie lifecycle works:

1. You log in to LeetCode in your browser
2. LeetCode's server validates your credentials and creates a **session** — a record stored on their servers that says "user X is authenticated"
3. The server sends back a response with a `Set-Cookie` header:

```
   Set-Cookie: LEETCODE_SESSION=eyJ0eXAiOiJKV1QiLCJhbGciOi...; HttpOnly; Secure
```

4. Your browser stores this cookie
5. On every subsequent request to `leetcode.com`, your browser automatically includes:

```
   Cookie: LEETCODE_SESSION=eyJ0eXAiOiJKV1QiLCJhbGciOi...
```

6. LC's server reads the session token, looks up the session, and knows who you are

Your Python code doesn't have a browser to manage cookies automatically. So you copy the `LEETCODE_SESSION` cookie value manually (from your browser's dev tools) and paste it here:

python

```python
LC_SESSION_COOKIE = "YOUR_LEETCODE_SESSION_COOKIE"  # line 20
```

Then pass it manually in every request:

python

```python
"cookie": f"LEETCODE_SESSION={LC_SESSION_COOKIE}",  # line 191, line 326
```

This is exactly what the browser would do — you're just doing it manually. The server can't tell the difference.

**How to get the cookie value:** Open LC in Chrome/Firefox → F12 → Application tab → Cookies → `leetcode.com` → find `LEETCODE_SESSION` → copy the value.

---

### Timeouts

python

```python
requests.post(url, ..., timeout=15)   # line 85
requests.get(url,  ..., timeout=12)   # line 87
```

When you open a TCP connection to a server, several things can go wrong silently:

- The server accepts the connection but never responds (hanging)
- A firewall drops the packets silently
- The server starts sending data then stalls halfway through
- DNS resolution takes forever

Without a timeout, `requests` would wait indefinitely in any of these cases. Your program would hang forever with no indication of what went wrong.

`timeout=N` tells `requests`: if you haven't received the full response within N seconds, raise a `requests.exceptions.Timeout` exception.

Technically, `timeout` in `requests` applies to each individual network operation (connect, read), not the total request time. But for practical purposes, it's the upper bound on how long any single call to `_req` will block waiting for the network.

The `Timeout` exception is a subclass of `requests.exceptions.RequestException`, which is itself a subclass of `Exception`, so it gets caught by:

python

```python
except Exception:           # line 96
    time.sleep(2 ** attempt)
```

And then retried.

POST gets `timeout=15` and GET gets `timeout=12`. POST is slightly more generous because LeetCode's GraphQL endpoint sometimes takes longer to respond (it's processing a query server-side, not just returning a cached resource).

---

### `raise_for_status()` and Exception Types

The full exception hierarchy in `requests`:

```
IOError
└── requests.exceptions.RequestException
    ├── requests.exceptions.ConnectionError
    │   └── requests.exceptions.ProxyError
    │   └── requests.exceptions.SSLError
    ├── requests.exceptions.Timeout
    │   ├── requests.exceptions.ConnectTimeout
    │   └── requests.exceptions.ReadTimeout
    ├── requests.exceptions.URLRequired
    ├── requests.exceptions.TooManyRedirects
    └── requests.exceptions.HTTPError          ← raised by raise_for_status()
```

Your code catches two levels:

python

```python
except requests.exceptions.HTTPError as e:    # line 90 — specifically HTTP errors
    sc = e.response.status_code
    if sc in (429, 500, 502, 503, 504):
        time.sleep(2 ** attempt)
    else:
        return None

except Exception:                             # line 96 — everything else
    time.sleep(2 ** attempt)
```

`HTTPError` is caught first because you need to read the status code to decide whether to retry. All other exceptions (connection failed, timeout, SSL error, etc.) fall into the generic `except Exception` — they're all treated the same: wait and retry.

---

### Retry Logic and Exponential Backoff

python

```python
def _req(url, method="GET", headers=None, json_data=None, retries=3):
    for attempt in range(retries):   # attempt = 0, 1, 2
        try:
            ...
            r.raise_for_status()
            return r.json()          # success — return immediately
        except requests.exceptions.HTTPError as e:
            sc = e.response.status_code
            if sc in (429, 500, 502, 503, 504):
                time.sleep(2 ** attempt)   # wait 1s, 2s, 4s
            else:
                return None
        except Exception:
            time.sleep(2 ** attempt)
    return None                      # all retries exhausted
```

**Why retry at all?**

Network calls fail for transient reasons all the time. A server might be momentarily overloaded. A packet might get dropped. Your DNS might hiccup. A single failure doesn't mean the server is genuinely down — it might be fine 500ms later. Retrying gives you resilience against these temporary glitches.

**Why exponential backoff?**

`2 ** attempt` gives you waits of: 1 second (attempt 0), 2 seconds (attempt 1), 4 seconds (attempt 2).

If you retry instantly, you're just hammering an already-overloaded server with more load. This makes the problem worse and gets you rate-limited harder. Exponential backoff gives the server breathing room between each attempt. The "exponential" part means the wait grows fast enough that even many simultaneous clients doing this won't collectively destroy the server.

The real-world standard (used in AWS, Google, etc.) often adds **jitter** — a random offset on top of the exponential delay — to prevent multiple clients from syncing up their retries. Your code doesn't do this (it doesn't need to at this scale), but it's the production extension of this pattern.

**What `return None` means downstream**

When all retries fail (or a non-retriable error occurs), `_req` returns `None`. Every caller checks for this:

python

```python
# cf_fetch_problems, line 114
if not pool_data or pool_data.get("status") != "OK":
    return [], []

# ac_fetch_problems, line 155
if not probs or not models:
    return [], []
```

`not None` is `True` in Python, so `if not pool_data` catches the `None` case. The function degrades gracefully — returns empty results rather than crashing.

---

### `response.json()` in depth

python

```python
return r.json()  # line 89
```

When a server sends JSON, the response body is a **string** that follows the JSON format. `r.json()` calls Python's built-in JSON parser on that string and gives you the native Python equivalent:

|JSON type|Python type|
|---|---|
|`object {}`|`dict`|
|`array []`|`list`|
|`string ""`|`str`|
|`number`|`int` or `float`|
|`true`/`false`|`True`/`False`|
|`null`|`None`|

So a Codeforces response like:

json

```json
{
  "status": "OK",
  "result": [
    {"contestId": 1900, "index": "A", "name": "Watermelon", "rating": 800}
  ]
}
```

Becomes this Python dict after `r.json()`:

python

```python
{
    "status": "OK",
    "result": [
        {"contestId": 1900, "index": "A", "name": "Watermelon", "rating": 800}
    ]
}
```

And then `cf_fetch_problems` navigates it:

python

```python
for p in pool_data["result"]["problems"]:   # line 118
    rating = p.get("rating")                # line 121
```

The entire data pipeline of your code — every `["result"]`, every `.get("status")`, every `for p in ...` — operates on the output of `r.json()`. That one call is the bridge between raw HTTP response and usable Python data.