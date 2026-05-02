## Topic 2: REST APIs

### What is an API?

Before "REST" — what is an API at the HTTP level?

**API** stands for Application Programming Interface. In the context of web services, an API is a server that exposes URLs you can make HTTP requests to, and returns structured data (almost always JSON) instead of HTML. That's the entire distinction between an API and a regular website — a website returns HTML meant for a browser to render visually, an API returns raw data meant for a program to process.

When your code hits `https://codeforces.com/api/problemset.problems`, Codeforces's server doesn't return a webpage. It returns a JSON object containing every problem in their problemset. Your code reads that JSON, filters it, and picks problems from it. That interaction — program talks to server, server returns structured data — is what "using an API" means.

---

### What is REST?

REST stands for **Representational State Transfer**. It's an architectural style — a set of conventions for how to design an HTTP API. It's not a protocol, not a library, not a specification you install. It's a design philosophy that most public APIs follow.

The conventions are:

#### Resources and URLs

In REST, everything is a **resource** — a noun that represents some piece of data. A resource is identified by a URL. The URL structure should reflect the hierarchy of the data.

Examples:

```
/api/users                    → the collection of all users
/api/users/123                → the specific user with id 123
/api/users/123/submissions    → all submissions belonging to user 123
/api/problemset/problems      → the collection of all problems
```

The URL identifies _what_ you're accessing. The HTTP method identifies _what operation_ you're performing on it.

#### HTTP Methods map to operations

|Method|Operation|
|---|---|
|GET|Read — retrieve the resource|
|POST|Create — submit new data|
|PUT|Update — replace the resource entirely|
|PATCH|Update — modify part of the resource|
|DELETE|Delete the resource|

Since your code is only reading data, every request is either GET or POST (POST because GraphQL, which we'll cover in Topic 3, always uses POST regardless of operation).

#### Statelessness

Every request must contain all the information the server needs to fulfill it. The server doesn't remember anything from your previous request. If you need authentication, you include credentials in _every_ request. This is why your LeetCode cookie goes in every single `_req` call — the server has no memory of the previous call.

#### Uniform interface

Clients interact with the API the same way regardless of what they're accessing — HTTP methods, standard status codes, and JSON responses. You don't need special protocols or libraries beyond plain HTTP.

---

### URL Structure

A URL has several components. Understanding each one is necessary for reading API docs and constructing requests correctly.

```
https://codeforces.com/api/user.status?handle=lambdadelta&from=1&count=10000
│       │               │              │
scheme  host            path           query string
```

#### Scheme

`https` — the protocol. `https` is HTTP over TLS (encrypted). All three platforms your code uses require HTTPS.

#### Host

`codeforces.com` — the domain name of the server. DNS resolves this to an IP address.

#### Path

`/api/user.status` — identifies the specific endpoint (resource) on the server. This is what the server's router uses to decide which handler function to run.

#### Query String

`?handle=lambdadelta&from=1&count=10000`

Everything after the `?` is the query string. It's a sequence of `key=value` pairs separated by `&`. This is how you pass parameters to a GET request — since GET has no body, all input goes in the URL.

Breaking it down:

- `handle=lambdadelta` — which user's submissions to fetch
- `from=1` — start index (1-based)
- `count=10000` — maximum number of submissions to return

Your code constructs these URLs with f-strings directly:

python

```python
# cf_fetch_problems, line 106
_req(f"https://codeforces.com/api/user.status?handle={CF_HANDLE}&from=1&count=10000")

# ac_fetch_problems, line 146
_req(f"https://kenkoooo.com/atcoder/atcoder-api/v3/user/submissions?user={AC_HANDLE}&from_second=0")
```

The `CF_HANDLE` and `AC_HANDLE` variables get interpolated directly into the query string. This works fine for simple cases. If handle values could contain special characters (spaces, `&`, `=`), you'd need URL encoding — but handles on these platforms are alphanumeric, so direct interpolation is safe.

An alternative is to use `requests`'s `params` argument, which handles URL encoding automatically:

python

```python
requests.get(
    "https://codeforces.com/api/user.status",
    params={"handle": CF_HANDLE, "from": 1, "count": 10000}
)
```

`requests` would build the same URL from that dict. Your code doesn't use this pattern, but it's the recommended approach when parameter values could contain special characters.

---

### Base URL and Endpoints

Every API has a **base URL** — the common prefix for all its endpoints. Then each endpoint is a specific path appended to that base.

**Codeforces:**

```
Base URL:  https://codeforces.com/api
Endpoints: /user.status
           /problemset.problems
           /contest.list
           /user.info
```

**Kenkoooo AtCoder:**

```
Base URL:  https://kenkoooo.com/atcoder
Endpoints: /atcoder-api/v3/user/submissions
           /resources/problems.json
           /resources/problem-models.json
```

Note that Kenkoooo has two different path prefixes (`/atcoder-api/v3/` and `/resources/`) — these are just different parts of their backend serving different kinds of data. The "resources" endpoints serve static JSON files that are periodically regenerated. The "atcoder-api" endpoints are dynamic.

---

### How to Read API Documentation

API docs tell you: what endpoints exist, what parameters they accept, and what the response looks like. Let's go through both APIs your code uses.

---

### The Codeforces API

Codeforces has official public API documentation at `https://codeforces.com/apiHelp`.

#### Authentication

The Codeforces API has two modes:

- **Public endpoints** — no authentication required. `problemset.problems` and `user.status` are both public.
- **Authenticated endpoints** — require an API key + secret + signature. Your code doesn't use these.

#### Endpoint: `user.status`

**Full URL pattern:**

```
https://codeforces.com/api/user.status?handle={handle}&from={from}&count={count}
```

**Parameters:**

|Parameter|Required|Type|Description|
|---|---|---|---|
|`handle`|Yes|string|The Codeforces user handle|
|`from`|No|int|1-based index of the first submission to return (default: 1)|
|`count`|No|int|Number of submissions to return (default: 10, max: 10000)|

Your code uses:

python

```python
f"https://codeforces.com/api/user.status?handle={CF_HANDLE}&from=1&count=10000"
```

`from=1&count=10000` fetches up to 10,000 most recent submissions starting from index 1 — effectively, all of them up to the limit.

**Response structure:**

json

```json
{
  "status": "OK",
  "result": [
    {
      "id": 123456789,
      "contestId": 1900,
      "problem": {
        "contestId": 1900,
        "index": "A",
        "name": "Watermelon",
        "rating": 800,
        "tags": ["math"]
      },
      "verdict": "OK",
      "creationTimeSeconds": 1700000000
    },
    ...
  ]
}
```

The top-level object always has `"status"` (`"OK"` or `"FAILED"`) and `"result"` (the data). When `status` is `"FAILED"`, there's a `"comment"` field instead of `"result"` explaining the error.

Your code reads this at lines 108–111:

python

```python
if solved_data and solved_data.get("status") == "OK":
    for s in solved_data["result"]:
        if s.get("verdict") == "OK" and "contestId" in s["problem"]:
            solved.add(f"{s['problem']['contestId']}-{s['problem']['index']}")
```

Breaking this down:

- `solved_data.get("status") == "OK"` — guards against the `"FAILED"` case
- `for s in solved_data["result"]` — iterates over the list of submissions
- `s.get("verdict") == "OK"` — `"OK"` verdict means accepted. Other verdicts are `"WRONG_ANSWER"`, `"TIME_LIMIT_EXCEEDED"`, `"RUNTIME_ERROR"`, etc.
- `"contestId" in s["problem"]` — some problems are from gym contests or other contexts and don't have a regular `contestId`. This guards against those.
- The solved set entry is `"1900-A"` — `contestId` hyphen `index`. This is your internal ID format used to match problems later.

#### Endpoint: `problemset.problems`

**Full URL:**

```
https://codeforces.com/api/problemset.problems
```

No required parameters. Optional parameters include `tags` (filter by tag) and `problemsetName` (specific problemset name). Your code doesn't use either — it fetches everything and filters locally.

**Response structure:**

json

```json
{
  "status": "OK",
  "result": {
    "problems": [
      {
        "contestId": 1900,
        "index": "A",
        "name": "Watermelon",
        "rating": 800,
        "tags": ["math", "brute force"]
      }
    ],
    "problemStatistics": [
      {
        "contestId": 1900,
        "index": "A",
        "solvedCount": 50000
      }
    ]
  }
}
```

Note the structure difference from `user.status`: here `result` is an **object** with two keys (`problems` and `problemStatistics`), not a list. `user.status` had `result` as a list directly.

Your code at line 118:

python

```python
for p in pool_data["result"]["problems"]:
```

This navigates: top-level dict → `"result"` → `"problems"` → list of problem objects.

#### Filtering logic in `cf_fetch_problems`

After fetching, your code filters the problem list against several criteria:

python

```python
# line 119
if p.get("contestId", 0) < CF_RECENT:
    continue
```

`CF_RECENT = 1900` — skips problems from contests older than #1900. `p.get("contestId", 0)` uses a default of `0` so that problems with no `contestId` (which would be `None` from `.get()`) compare safely — `None < 1900` would raise a `TypeError` in Python 3, but `0 < 1900` is fine.

python

```python
# line 121-122
rating = p.get("rating")
if rating is None:
    continue
```

Some problems don't have a rating (unrated contests). These are skipped entirely.

python

```python
# line 124-125
pid = f"{p['contestId']}-{p['index']}"
if pid in solved:
    continue
```

The `solved` set was built from `user.status`. The problem ID format `"1900-A"` matches exactly — same construction on both sides.

python

```python
# line 127
if tags and not set(tags).intersection(set(p.get("tags", []))):
    continue
```

If the user specified tags (e.g., `["dp", "graphs"]`), skip problems that don't have any of those tags. `intersection` checks for _any_ overlap — a problem with tags `["dp", "math"]` would pass a filter of `["dp"]`.

---

### The Kenkoooo AtCoder API

Kenkoooo is a community-maintained project that aggregates AtCoder data. It's not an official AtCoder API. The source is open at `https://github.com/kenkoooo/AtCoderProblems`.

#### Authentication

None required. It's a public API. However, it does check `User-Agent` (which is why your code sends `"Mozilla/5.0"`).

#### Endpoint: `user/submissions`

```
https://kenkoooo.com/atcoder/atcoder-api/v3/user/submissions?user={user}&from_second={from_second}
```

**Parameters:**

|Parameter|Required|Type|Description|
|---|---|---|---|
|`user`|Yes|string|AtCoder username|
|`from_second`|Yes|int|Unix timestamp — return submissions from this time onwards. `0` means all submissions.|

Your code:

python

```python
f"https://kenkoooo.com/atcoder/atcoder-api/v3/user/submissions?user={AC_HANDLE}&from_second=0"
```

`from_second=0` fetches every submission ever — Unix timestamp 0 is January 1, 1970.

**Response structure:**

json

```json
[
  {
    "id": 12345678,
    "epoch_second": 1700000000,
    "problem_id": "abc300_a",
    "contest_id": "abc300",
    "user_id": "rawmeat21",
    "language": "C++ (GCC 9.2.1)",
    "point": 100.0,
    "length": 512,
    "result": "AC",
    "execution_time": 2
  },
  ...
]
```

Note: the top level is a **list**, not an object with a `"status"` field. There's no `"status": "OK"` wrapper here unlike Codeforces. If the request fails, you get an HTTP error status code, not a JSON error object. That's why `_req` handles it through `raise_for_status()` and your code just checks `if solved_data:` rather than checking a status field.

Your code at lines 150–151:

python

```python
solved = {s["problem_id"] for s in solved_data if s.get("result") == "AC"}
```

This is a **set comprehension** — it builds a set of `problem_id` strings for all submissions where the result was `"AC"` (accepted).

#### Endpoint: `resources/problems.json`

```
https://kenkoooo.com/atcoder/resources/problems.json
```

No parameters. Returns a static JSON file listing every problem in AtCoder's database.

**Response structure:**

json

```json
[
  {
    "id": "abc300_a",
    "contest_id": "abc300",
    "problem_index": "A",
    "name": "Poisonous Ants",
    "title": "A. Poisonous Ants"
  },
  ...
]
```

Again, a top-level list. Each entry has an `id` (the problem's unique identifier — used as the key across all Kenkoooo data), `contest_id`, and `title`.

#### Endpoint: `resources/problem-models.json`

```
https://kenkoooo.com/atcoder/resources/problem-models.json
```

Returns difficulty ratings for every problem. This is a **different structure** from the others — it's a JSON object where each key is a `problem_id` and the value is the model data.

**Response structure:**

json

```json
{
  "abc300_a": {
    "slope": -0.00123,
    "intercept": 1234.5,
    "variance": 0.456,
    "difficulty": 312,
    "discrimination": 0.789,
    "irt_loglikelihood": -100.0,
    "irt_users": 5000,
    "is_experimental": false
  },
  "abc300_b": {
    "difficulty": 487,
    ...
  },
  ...
}
```

This is a **dict** at the top level, not a list. The problem ID is the key. That's why your code uses:

python

```python
# ac_fetch_problems, line 170
model = models.get(pid)
```

`.get(pid)` looks up the problem ID as a key in this dict. If the problem has no model (some problems don't), it returns `None`.

#### How `ac_fetch_problems` joins these two datasets

This is an important pattern worth understanding fully. You have two separate API responses:

- `probs` — list of all problems (has `id`, `contest_id`, `title` but no difficulty)
- `models` — dict of problem difficulties keyed by problem ID

You need to combine them to get problems with both metadata and difficulty. This is essentially a **join** operation — same concept as SQL JOIN:

python

```python
for p in probs:                    # iterate the problems list
    pid = p["id"]                  # get the problem ID
    ...
    model = models.get(pid)        # look up difficulty by ID — O(1) dict lookup
    if not model or model.get("difficulty") is None:
        continue                   # skip problems with no difficulty model
    diff = model["difficulty"]
```

`models` is used as a lookup table. For each problem in `probs`, you do a dict lookup into `models` to find its difficulty. This is O(1) per lookup, O(n) total — efficient.

#### Recency filtering for AtCoder

python

```python
# lines 163-168
parts = pid.split("_")
if parts and parts[0][:3] in ("abc", "arc", "agc"):
    try:
        if int(parts[0][3:]) < AC_RECENT:
            continue
    except ValueError:
        pass
```

AtCoder problem IDs follow the pattern `abc300_a` — contest prefix + number + underscore + problem letter. `AC_RECENT = 380` means: only include problems from ABC/ARC/AGC contest #380 onwards.

`pid.split("_")` on `"abc300_a"` gives `["abc300", "a"]`. `parts[0]` is `"abc300"`. `parts[0][:3]` is `"abc"`. `parts[0][3:]` is `"300"`, converted to int for comparison.

The `try/except ValueError` handles contest IDs that don't follow the numeric convention (some special contests don't).

---

### Error Handling for API Responses

The two APIs handle errors differently, and your code handles each appropriately.

#### Codeforces error handling

CF wraps errors in JSON with a `"status": "FAILED"` field:

json

```json
{
  "status": "FAILED",
  "comment": "handles: User with handle lambdadelta not found"
}
```

Your code checks:

python

```python
# line 108, 114
if solved_data and solved_data.get("status") == "OK":
if not pool_data or pool_data.get("status") != "OK":
```

The double-check: first `solved_data` (truthy check — is it not `None`?), then `.get("status") == "OK"` (did the API itself report success?). If either fails, the function degrades gracefully.

#### Kenkoooo error handling

Kenkoooo returns HTTP error status codes on failure (no JSON error wrapper). Since `_req` calls `raise_for_status()`, a failed Kenkoooo request results in `_req` returning `None`. Your code checks:

python

```python
# line 155
if not probs or not models:
    return [], []
```

`not None` is `True`, so `if not probs` catches the case where `_req` returned `None`.

---

### URL construction summary across your code

|Location|URL Pattern|Method|Parameters|
|---|---|---|---|
|`cf_fetch_problems` line 106|`codeforces.com/api/user.status`|GET|`handle`, `from`, `count` in query string|
|`cf_fetch_problems` line 113|`codeforces.com/api/problemset.problems`|GET|none|
|`_cf_solved_set` line 300|`codeforces.com/api/user.status`|GET|same as above|
|`ac_fetch_problems` line 146|`kenkoooo.com/.../user/submissions`|GET|`user`, `from_second` in query string|
|`ac_fetch_problems` line 153|`kenkoooo.com/.../problems.json`|GET|none|
|`ac_fetch_problems` line 154|`kenkoooo.com/.../problem-models.json`|GET|none|
|`_ac_solved_set` line 311|`kenkoooo.com/.../user/submissions`|GET|same as above|
|`lc_fetch_problems` line 210|`leetcode.com/graphql`|POST|GraphQL query + variables in JSON body|
|`_lc_solved_set` line 376|`leetcode.com/graphql`|POST|GraphQL query + variables in JSON body|