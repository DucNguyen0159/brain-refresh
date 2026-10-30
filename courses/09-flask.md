# Flask

Course 9 of 15. Previous: [08-python.md](08-python.md). Next: [10-sql.md](10-sql.md).

Flask maps a URL and an HTTP method to a Python function. You install it with `pip install flask`. A small app is one file. A larger app uses blueprints and a factory.

Every section has the same parts: the concept, the terms, the methods, several cases with what the response is, common mistakes, a tip, and a practice with a solution.

## Contents

1. [An application and a route](#1-an-application-and-a-route)
2. [Request data](#2-request-data)
3. [Responses and status codes](#3-responses-and-status-codes)
4. [Templates](#4-templates)
5. [Static files, config, and sessions](#5-static-files-config-and-sessions)
6. [Blueprints and errors](#6-blueprints-and-errors)

## 1. An application and a route

### Concept

`Flask(__name__)` builds the application. `__name__` helps Flask find templates and static files next to that module. A decorator registers a function for a path. `@app.get("/health")` is `GET /health`. `@app.route("/homes", methods=["GET", "POST"])` allows several methods on one path. Returning a dict makes a JSON response. `debug=True` reloads on save and shows a debugger. Use it in development only.

A variable in the path becomes a function argument. The default converter is a string without a slash. `<int:year>` is an integer. A value that does not match the converter does not match the route.

### Terms

| Piece | Meaning |
| --- | --- |
| `Flask(__name__)` | The application object |
| Route | A path plus the methods that call one function |
| `@app.get("/health")` | Register this function for `GET /health` |
| `@app.route(..., methods=[...])` | Several methods on one path |
| Path variable | `/homes/<home_id>` passes `home_id` into the function |
| Converter | A rule for that variable, such as `int` or `path` |
| `debug=True` | Reloads on save and shows a debugger. Development only |
| `app.run` | The development server. Not the server you use under load |

| Converter | Accepts |
| --- | --- |
| default `string` | Text without a slash |
| `int` | An integer. A non-integer does not match the route |
| `float` | A float |
| `path` | Text that may contain slashes |
| `uuid` | A UUID |

### Methods

| Call | What the client gets |
| --- | --- |
| `@app.get("/health")` | Only `GET` |
| `@app.post("/homes")` | Only `POST` |
| `@app.route("/homes", methods=["GET", "POST"])` | `GET` and `POST` |
| `return {"status": "ok"}` | JSON, status 200 |
| `app.run(debug=True, port=5000)` | Development server on port 5000, and only when this file is the program if you guard it with `__name__` |

### Examples

**Case: `GET /health` returns JSON**

```python
from flask import Flask

app = Flask(__name__)

@app.get("/health")
def health():
    return {"status": "ok"}
```

Result: `GET /health` returns status 200 and the JSON body `{"status": "ok"}`.

Why: a dict return value is encoded as JSON. The decorator registers `GET` only.

**Case: one path, two methods**

```python
@app.route("/homes", methods=["GET", "POST"])
def homes():
    return {"ok": True}
```

Result: `GET /homes` and `POST /homes` each return status 200 and `{"ok": true}`. A `PUT` to that same path does not run `homes`.

Why: `methods` is the allow-list. Flask still finds the path, and the method is not on the list, so the response is 405.

**Case: a string variable and an int converter**

```python
@app.get("/homes/<home_id>")
def home(home_id):
    return {"id": home_id}

@app.get("/reports/<int:year>")
def report(year):
    return {"year": year}
```

Result: `GET /homes/R1` returns `{"id": "R1"}` and `home_id` is a string. `GET /reports/2026` returns `{"year": 2026}` and `year` is an int. `GET /reports/nope` does not match `report`.

Why: the default converter stops at a slash and leaves the value as text. `int` matches only an integer. A non-integer never calls the function.

**Case: the development server starts from the main guard**

```python
if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

Result: `python3 app.py` listens on port 5000 and reloads when you save. Importing the module does not call `app.run`.

Why: the guard is true only when this file is the program. `debug=True` is the reloader and the debugger. Turn it off for any server other people can reach.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `debug=True` on a public server | The debugger can execute code | `debug=False`, and run under a WSGI server |
| Assuming `<home_id>` is an int | It is a string. `"10"` and `10` compare unequal | Use `<int:home_id>` when you require an integer |
| A second route with the same path and method | One of them is hidden | One function per path and method |
| Calling `app.run` at import time | Importing the module starts a server | Put `app.run` under `if __name__ == "__main__"` |

### Tip

`__name__` is doing two jobs: Flask uses it to locate `templates` and `static`, and the main guard uses it to decide whether to listen.

### Practice

Register `GET /health` so the JSON body is `{"status": "ok"}`. Say what `GET /reports/nope` does when the only year route uses `<int:year>`.

### Solution

```python
from flask import Flask

app = Flask(__name__)

@app.get("/health")
def health():
    return {"status": "ok"}

@app.get("/reports/<int:year>")
def report(year):
    return {"year": year}
```

Result: `GET /health` is status 200 and `{"status": "ok"}`. `GET /reports/nope` does not match `report`, so Flask does not call it.

## 2. Request data

### Concept

The client can send any body. Validate on the server. `request.args` is the query string, `?city=Irvine`. `request.form` is a form body. `request.json` is the parsed JSON body. `request.get_json(silent=True)` returns `None` instead of failing when the body is missing or is not JSON. `.get` returns `None` when the key is missing, or the default you pass. Indexing `request.args["city"]` raises when the key is missing.

A view can return a body, `(body, status)`, or `(body, status, headers)`.

### Terms

| Term | Meaning |
| --- | --- |
| Query string | The `?city=Irvine` part of the URL. `request.args` |
| Form body | `application/x-www-form-urlencoded` or `multipart/form-data`. `request.form` |
| JSON body | `application/json`. `request.json`, or `request.get_json()` |
| Header | `request.headers.get("Authorization")` |
| Upload | `request.files["photo"]` on a multipart form |
| Path converter | The function argument, not a part of `request.args` |
| `silent=True` | `get_json` returns `None` instead of raising |

| | `request.args` | `request.form` | `request.json` |
| --- | --- | --- | --- |
| Where the data is | The query string, `?city=Irvine` | A form body | A JSON body. `request.get_json()` reads the same body |
| Typical method | Any, often `GET` | `POST` | `POST` or `PUT` |
| Content type | None. It is in the URL | `application/x-www-form-urlencoded` or `multipart/form-data` | `application/json` |
| One value | `.get("city")` | `.get("city")` | The dict's `.get("city")` after `get_json` |
| Several values for one key | `.getlist("city")` | `.getlist("city")` | A JSON list under that key |
| Missing key | `.get` returns `None`. `["city"]` raises | Same | `.get` returns `None` when the body parsed to a dict. A missing body returns `None` when `silent=True` |
| A file | No | `request.files["photo"]` when the form is multipart | No |

### Methods

| Source | API |
| --- | --- |
| Query string `?city=Irvine` | `request.args.get("city")` |
| Form body | `request.form.get("city")` |
| JSON body | `request.json` or `request.get_json()` |
| JSON body that might be absent | `request.get_json(silent=True) or {}` |
| Header | `request.headers.get("Authorization")` |
| Uploaded file | `request.files["photo"]` |
| Path converter | The function argument |

| Return | Status | Body | Headers you set |
| --- | --- | --- | --- |
| `"hello"` | 200 | Text | Flask picks the content type |
| `{"ok": True}` | 200 | JSON | Flask picks JSON |
| `(body, 201)` | The number you gave | `body` | Flask picks from the body type |
| `(body, 201, {"X-Request": "1"})` | 201 | `body` | Those, plus what Flask adds |
| `jsonify(obj)` | 200 unless you change the response | JSON | You can set more on the returned response |
| `abort(404)` | 404 | The error handler's body | The error handler's |

### Examples

**Case: a query parameter**

```python
from flask import request

@app.get("/homes")
def homes():
    city = request.args.get("city", "")
    return {"city": city}
```

Result: `GET /homes?city=Irvine` returns `{"city": "Irvine"}`. `GET /homes` returns `{"city": ""}`.

Why: `args` reads the query string. The second argument of `.get` is the default when `city` is absent.

**Case: indexing a missing query key**

```python
@app.get("/homes")
def homes():
    city = request.args["city"]
    return {"city": city}
```

Result: `GET /homes?city=Irvine` returns `{"city": "Irvine"}`. `GET /homes` raises. The view does not return a body you chose.

Why: `["city"]` requires the key. `.get("city")` returns `None` instead. The same split applies to `request.form`.

**Case: a JSON body, validated**

```python
@app.post("/homes")
def create_home():
    data = request.get_json(silent=True) or {}
    city = data.get("city", "")
    if not isinstance(city, str) or not city.strip():
        return {"error": "city is required"}, 400
    return {"city": city.strip()}, 201
```

Result: `POST /homes` with `Content-Type: application/json` and body `{"city": " Irvine "}` returns status 201 and `{"city": "Irvine"}`. A missing body, a non-JSON body, a missing `city`, or a blank `city` returns status 400 and `{"error": "city is required"}`.

Why: `get_json(silent=True)` is `None` when the body is missing or is not JSON, and `or {}` turns that into a dict. `request.json` is the parsed body when the call is allowed to fail instead. The client can send any JSON type, so check `isinstance` before calling `strip`.

**Case: the same name in three places**

```python
@app.route("/city", methods=["GET", "POST"])
def city():
    from_query = request.args.get("city")
    from_form = request.form.get("city")
    payload = request.get_json(silent=True) or {}
    from_json = payload.get("city")
    return {
        "query": from_query,
        "form": from_form,
        "json": from_json,
    }
```

Result: `GET /city?city=Irvine` fills `query` and leaves `form` and `json` as JSON `null`. A form POST of `city=Tustin` fills `form`. A JSON POST of `{"city": "Anaheim"}` fills `json`. The other two stay `null` unless that request also sent them.

Why: `request.args`, `request.form`, and `request.json` are different bags. A field named `city` in one of them is not automatically copied into the others.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `request.args["city"]` when the key is optional | The view raises | `.get("city", "")` |
| Reading JSON from `request.form` | The form dict is empty. The body was JSON | `request.get_json(silent=True)` or `request.json` |
| Calling `.strip()` on a JSON number | `AttributeError` | `isinstance(city, str)` first |
| Trusting the browser to validate | A client can skip your form and send any body | Check presence and type on the server |

### Tip

Name the bag in the code you are reading. Query, form, and JSON can all carry a key called `city`, and they do not share storage.

### Practice

Write a `POST /homes` view that reads JSON with `silent=True`, returns 400 when `city` is missing or blank, and returns 201 with the stripped city otherwise.

### Solution

```python
@app.post("/homes")
def create_home():
    data = request.get_json(silent=True) or {}
    city = data.get("city", "")
    if not isinstance(city, str) or not city.strip():
        return {"error": "city is required"}, 400
    return {"city": city.strip()}, 201
```

Result: body `{"city": " Irvine "}` returns status 201 and `{"city": "Irvine"}`. Body `{}` returns status 400 and `{"error": "city is required"}`.

## 3. Responses and status codes

### Concept

The return value of a view is the response. A string is text. A dict or a list is JSON. A tuple sets the status and, when a third item is present, extra headers. `jsonify` builds a JSON response you can keep editing. `redirect` sends the client to another URL. `abort` raises an HTTP exception. An error handler can turn that exception into a page or JSON.

### Terms

| Status | When |
| --- | --- |
| 200 | The read or update worked |
| 201 | Created |
| 204 | Success, no body |
| 301 / 302 | Moved. `redirect` uses 302 |
| 400 | Bad input |
| 401 | Not authenticated |
| 403 | Authenticated, not allowed |
| 404 | No such resource |
| 415 | Unsupported media type |
| 500 | Your code failed |

| Return | What Flask sends |
| --- | --- |
| `"hello"` | Text, status 200 |
| `{"ok": True}` | JSON, status 200 |
| `({"error": "missing"}, 400)` | JSON and that status |
| `jsonify(obj)` | JSON, and you can keep building the response. Status 200 unless you set it |
| `redirect(url)` | A redirect response. Status 302 |
| `send_file(path)` | A file download or inline file |
| `make_response` | A response object you can set headers on |
| `abort(404)` | An HTTP exception. Status 404 |

### Methods

| Call | Effect |
| --- | --- |
| `return "hello"` | Body `hello`, status 200 |
| `return {"ok": True}` | JSON body, status 200 |
| `return {"id": 1}, 201` | JSON body, status 201 |
| `return body, 201, {"X-Request": "1"}` | Status 201 and that header |
| `response = jsonify(obj)` then `response.status_code = 201` | JSON, status 201 |
| `return redirect(url_for("index"))` | 302 and a `Location` header |
| `abort(404)` | Stop the view and raise the HTTP exception |
| `send_file(path)` | Send the bytes of a file |
| `make_response(body, status)` | A response you can attach headers to before returning |

### Examples

**Case: text, JSON, and a created status**

```python
@app.get("/text")
def text():
    return "hello"

@app.get("/data")
def data():
    return {"ok": True}

@app.post("/homes")
def create():
    return {"id": 1}, 201
```

Result: `GET /text` is status 200 and the body `hello`. `GET /data` is status 200 and `{"ok": true}`. `POST /homes` is status 201 and `{"id": 1}`.

Why: Flask picks the content type from the object you return. The second item of the tuple replaces the default status 200.

**Case: `jsonify` so you can set a header**

```python
from flask import jsonify

@app.post("/homes")
def create():
    response = jsonify({"ok": True})
    response.status_code = 201
    response.headers["X-Request"] = "1"
    return response
```

Result: status 201, JSON body `{"ok": true}`, and a header `X-Request: 1`.

Why: `jsonify` returns a response object. The status stays 200 until you change it. The three-item tuple `(body, 201, {"X-Request": "1"})` sets the same status and header without calling `jsonify` yourself.

**Case: `abort` when the row is missing**

```python
from flask import abort

@app.get("/homes/<home_id>")
def one_home(home_id):
    row = find_home(home_id)
    if row is None:
        abort(404)
    return row
```

Result: when `find_home` returns `None`, the client gets status 404. The lines after `abort` do not run. When `find_home` returns a dict, the client gets that dict as JSON with status 200.

Why: `abort` raises. An error handler can replace the default 404 body with a page or with JSON. An empty 200 would look like success.

**Case: a redirect**

```python
from flask import redirect, url_for

@app.post("/city")
def save_city():
    return redirect(url_for("index"))
```

Result: status 302 and a `Location` header whose path is the URL of the `index` function.

Why: `redirect` uses 302. `url_for` builds the path from the function name, so a later change to the route string updates this header.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Returning `None` for a missing row | The client gets status 200 and an empty body | `abort(404)` or `return {"error": "not found"}, 404` |
| Setting `status_code` after a plain dict return | The dict is already the return value. The assignment never happens | `jsonify`, or return `(body, status)` |
| Using 200 for bad input | The client treats the call as success | Return 400 with an error body |
| 401 and 403 swapped | A logged-out user is told they are forbidden, or a logged-in user is told to sign in | 401 means not authenticated. 403 means authenticated and not allowed |

### Tip

Pick the status before you write the body. The status is how the client decides whether to trust the JSON.

### Practice

Write `GET /homes/<home_id>` that calls `abort(404)` when `find_home(home_id)` returns `None`, and returns the row otherwise.

### Solution

```python
from flask import abort

@app.get("/homes/<home_id>")
def one_home(home_id):
    row = find_home(home_id)
    if row is None:
        abort(404)
    return row
```

Result: a missing id produces status 404. A found dict is JSON with status 200.

## 4. Templates

### Concept

Flask looks for HTML files in a `templates` folder next to the app. The language inside them is Jinja. `render_template` reads a file and returns HTML. `{{ name }}` prints a value and escapes it, so a `<` in the value stays text. `{% %}` is a tag such as `if` or `for`. `{{ value | safe }}` turns escaping off. Use it only for HTML you generated yourself. Autoescaping is why user text goes in `{{ }}` and not in `| safe`.

`{% extends "base.html" %}` fills blocks in a parent. `{% block body %}{% endblock %}` is a hole the child fills. `url_for("index")` builds a URL from a function name, so the route string can change. `url_for("home", home_id="R1")` fills a path variable. `url_for("static", filename="app.css")` points at the static folder.

### Terms

| Jinja | Meaning |
| --- | --- |
| `{{ name }}` | Print the value, escaped. A `<` in the value stays text |
| `{% if n %}...{% endif %}` | A branch |
| `{% for item in items %}...{% endfor %}` | A loop |
| `{{ value \| safe }}` | Do not escape. Only for HTML you generated yourself |
| `{% extends "base.html" %}` | This file fills blocks in a parent |
| `{% block body %}{% endblock %}` | A hole the child fills |
| `url_for("index")` | Build a URL from a function name |
| `url_for("home", home_id="R1")` | Fill a path variable |
| `url_for("static", filename="app.css")` | The static folder |

### Methods

| Call | Result |
| --- | --- |
| `render_template("index.html", cities=["Irvine", "Tustin"])` | HTML with `cities` available in the template |
| `{{ city }}` | Escaped text |
| `{{ city \| safe }}` | Raw HTML |
| `url_for("home", home_id="R1")` | The path of that route, with `R1` filled in |
| `url_for("static", filename="app.css")` | `/static/app.css` |

### Examples

**Case: a template loop**

```python
from flask import render_template

@app.get("/")
def index():
    return render_template("index.html", cities=["Irvine", "Tustin"])
```

```html
<h1>Cities</h1>
<ul>
  {% for city in cities %}
    <li>{{ city }}</li>
  {% endfor %}
</ul>
```

Result: `GET /` returns HTML. The list contains `<li>Irvine</li>` and `<li>Tustin</li>`.

Why: the keyword argument `cities` becomes the Jinja variable. The `{% for %}` tag repeats the `<li>`. `{{ city }}` prints each name.

**Case: user text is escaped**

```html
<p>{{ city }}</p>
```

Result: when `city` is `<b>Irvine</b>`, the HTML source contains `&lt;b&gt;Irvine&lt;/b&gt;`. The browser shows the characters `<b>Irvine</b>` as text.

Why: autoescaping treats the value as text. A `<` from a user cannot open a tag.

**Case: `| safe` marks HTML you produced**

```html
<p>{{ city | safe }}</p>
```

Result: when `city` is `<b>Irvine</b>` and you marked it safe, the browser renders the word Irvine in bold.

Why: `| safe` turns escaping off. That is right for a fragment your own code built, and wrong for text that arrived in a request.

**Case: `url_for` in the template**

```python
@app.get("/homes/<home_id>")
def home(home_id):
    return render_template("home.html", home_id=home_id)
```

```html
<a href="{{ url_for('home', home_id='R1') }}">R1</a>
<link rel="stylesheet" href="{{ url_for('static', filename='app.css') }}">
```

Result: the anchor's `href` is `/homes/R1`. The stylesheet `href` is `/static/app.css`.

Why: `url_for` uses the function name `home` and the converter name `home_id`. The static endpoint is built in. Hard-coding `/homes/R1` goes stale when the route changes.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `{{ city \| safe }}` on request data | The browser runs tags the user typed | Leave the default escaping on |
| A template saved outside `templates` | `render_template` raises `TemplateNotFound` | Put the file in the `templates` folder next to the app |
| `url_for("home.html")` | That is a file name, not an endpoint | Pass the function name: `url_for("home", home_id="R1")` |
| Writing `{% for %}` and forgetting `{% endfor %}` | The template fails to render | Close `if` with `endif` and `for` with `endfor` |

### Tip

If you can point at the Python function that produced a piece of HTML, `| safe` can be appropriate. If the string came from `request`, print it with `{{ }}`.

### Practice

Render `index.html` with `cities=["Irvine", "Tustin"]` and a template that lists each city in an `<li>`. Say what the page does with a city value of `<b>`.

### Solution

```python
@app.get("/")
def index():
    return render_template("index.html", cities=["Irvine", "Tustin"])
```

```html
<ul>
  {% for city in cities %}
    <li>{{ city }}</li>
  {% endfor %}
</ul>
```

Result: the HTML contains `<li>Irvine</li>` and `<li>Tustin</li>`. A city value `<b>` is escaped, so the page shows the characters `<b>` and does not create a bold element.

## 5. Static files, config, and sessions

### Concept

`static/app.css` is served as `/static/app.css`. `templates/index.html` is not served directly. `render_template` reads it. `app.config` holds settings. `SECRET_KEY` signs the session cookie. Without it, sessions refuse to work. Read it from the environment. Do not commit it. `MAX_CONTENT_LENGTH` rejects huge uploads.

`session` is a signed cookie. It is small, and it survives later requests from that browser. `g` is data for the current request only. A global variable is shared by every request in the process, so one user's data can leak to another.

The development server is `flask run` or `app.run`. It is not the server you use under load. A production process uses a WSGI server and points it at `app`.

### Terms

| Store | Lifetime |
| --- | --- |
| `session` | A signed cookie. Small. Survives requests from that browser |
| `g` | Data for the current request only |
| A global variable | Shared by every request in the process |
| `app.config` | Settings for this application object |
| `SECRET_KEY` | The key that signs the session cookie |
| `MAX_CONTENT_LENGTH` | The largest request body you accept, in bytes |

| Folder | Served as |
| --- | --- |
| `static/app.css` | `/static/app.css` |
| `templates/index.html` | Not served directly. `render_template` reads it |

### Methods

| Call | Effect |
| --- | --- |
| `app.config["SECRET_KEY"] = os.environ["SECRET_KEY"]` | Reads the signing key from the environment. A missing variable raises `KeyError` |
| `app.config["MAX_CONTENT_LENGTH"] = 1024 * 1024` | Rejects a body larger than 1 MiB |
| `session["city"] = request.form.get("city", "")` | Stores a value in the signed cookie |
| `redirect(url_for("index"))` | Sends the browser to the `index` route after the write |
| `g.user = "ada"` | Visible until this request ends |
| `flask run` or `app.run` | Development server |

### Examples

**Case: static is a URL, a template is not**

```text
static/app.css
templates/index.html
```

Result: a browser can `GET /static/app.css` and receive the file. `GET /templates/index.html` is not a route Flask adds for you. The template is used only when a view calls `render_template`.

Why: the `static` folder has a URL prefix. The `templates` folder is input to Jinja.

**Case: config from the environment**

```python
import os

app.config["SECRET_KEY"] = os.environ["SECRET_KEY"]
app.config["MAX_CONTENT_LENGTH"] = 1024 * 1024
```

Result: session cookies can be signed. A body larger than `1024 * 1024` bytes is rejected. If `SECRET_KEY` is unset, this line raises `KeyError` at startup.

Why: `os.environ["SECRET_KEY"]` reads the process environment. The file that holds the value stays out of the source you commit. `MAX_CONTENT_LENGTH` is the upload cap.

**Case: a session value and a redirect**

```python
from flask import redirect, request, session, url_for

@app.post("/city")
def save_city():
    session["city"] = request.form.get("city", "")
    return redirect(url_for("index"))
```

Result: a form POST with `city=Irvine` stores `Irvine` in the signed cookie and responds with status 302 to the `index` URL. The next request from that browser can read `session["city"]`. Setting `session` without `SECRET_KEY` fails.

Why: the cookie is signed so the client cannot forge a value that Flask will trust. It is still the client's storage, and it should stay small.

**Case: `g` ends with the request**

```python
from flask import g

@app.get("/who")
def who():
    g.user = "ada"
    return {"user": g.user}
```

Result: `GET /who` returns `{"user": "ada"}`. After the response is sent, `g.user` is gone. A module-level `current_user = "ada"` would still be there for the next request in the same process.

Why: `g` is request storage. A global is process storage. Two overlapping requests would share the global and can leak one user's data into the other response.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A committed `SECRET_KEY` | Anyone who can read the repo can forge session cookies | Read it from the environment |
| Putting a large object in `session` | The cookie grows, and browsers will drop it | Store an id in the cookie and the object on the server |
| Remembering a user in a global | The next request can see the previous user | Use `session` or `g` |
| Serving the site with `app.run` under load | One development process handles the traffic | Point a WSGI server at `app` |

### Tip

`session` answers "what should the next request from this browser remember?" `g` answers "what did this request already look up?"

### Practice

Set `SECRET_KEY` from the environment and `MAX_CONTENT_LENGTH` to `1024 * 1024`. In `POST /city`, save `request.form.get("city", "")` into `session` and redirect to `index`.

### Solution

```python
import os
from flask import redirect, request, session, url_for

app.config["SECRET_KEY"] = os.environ["SECRET_KEY"]
app.config["MAX_CONTENT_LENGTH"] = 1024 * 1024

@app.post("/city")
def save_city():
    session["city"] = request.form.get("city", "")
    return redirect(url_for("index"))
```

Result: a form field `city=Irvine` is stored in the signed cookie, and the response is a 302 to the `index` route. A body larger than 1 MiB is rejected.

## 6. Blueprints and errors

### Concept

A blueprint is a group of routes you can mount. `app.register_blueprint(homes, url_prefix="/homes")` puts those routes under `/homes`. `url_for("homes.list_homes")` uses the blueprint name and the function name.

`@app.errorhandler(404)` and `@app.errorhandler(500)` turn those statuses into a body you choose. Check the types and presence of JSON fields, because `get_json` may be `None`. Return 404 for a missing id. An empty 200 looks like success. Keep `debug` off in production, because the debugger can execute code. Log the exception, and send the client a short message. The stack is for you.

One app factory, `create_app()`, lets tests build an app with different config. A test client calls routes without opening a port.

### Terms

| Term | Meaning |
| --- | --- |
| Blueprint | A named group of routes, registered later |
| `url_prefix` | The path added in front of every route on that blueprint |
| Endpoint | `homes.list_homes`, the blueprint name, a dot, and the function name |
| Error handler | A function Flask calls for a status or an exception class |
| App factory | `create_app()` returns a configured application |
| Test client | Calls routes in-process. No listening port |

| Practice | Why |
| --- | --- |
| Check types and presence of JSON fields | `get_json` may be `None` |
| Return 404 for a missing id | An empty 200 looks like success |
| Keep `debug` off in production | The debugger can execute code |
| Log the exception, send a short message | The stack is for you |
| One app factory `create_app()` | Tests can build an app with different config |

### Methods

| Call | Effect |
| --- | --- |
| `Blueprint("homes", __name__)` | A blueprint named `homes` |
| `@homes.get("/")` | A route on the blueprint, not yet on the app |
| `app.register_blueprint(homes, url_prefix="/homes")` | Mounts it. The route is `GET /homes/` |
| `url_for("homes.list_homes")` | Builds that URL |
| `@app.errorhandler(404)` | Replaces the default 404 body |
| `app.config.from_mapping(SECRET_KEY=os.environ.get("SECRET_KEY", "dev"))` | Sets config. The `"dev"` fallback is only for a local test process |
| `create_app().test_client()` | A client whose `get` returns a response object |
| `response.get_json()` | The parsed JSON body |
| `response.status_code` | The status integer |

### Examples

**Case: a blueprint mounted at a prefix**

```python
from flask import Blueprint

homes = Blueprint("homes", __name__)

@homes.get("/")
def list_homes():
    return []

app.register_blueprint(homes, url_prefix="/homes")
```

Result: `GET /homes/` returns status 200 and JSON `[]`. `url_for("homes.list_homes")` is `/homes/`.

Why: the blueprint name is the first half of the endpoint. The prefix is added when you register, not when you write `@homes.get("/")`.

**Case: JSON bodies for 404 and 500**

```python
@app.errorhandler(404)
def missing(_error):
    return {"error": "not found"}, 404

@app.errorhandler(500)
def failed(_error):
    return {"error": "failed"}, 500
```

Result: an unknown path returns status 404 and `{"error": "not found"}`. An uncaught exception in a view returns status 500 and `{"error": "failed"}`.

Why: the handler's return uses the same tuple form as a normal view. Log the real exception before you replace a 500 body. The client gets the short message.

**Case: a factory and a test client**

```python
import os
from flask import Flask

def create_app():
    app = Flask(__name__)
    app.config.from_mapping(SECRET_KEY=os.environ.get("SECRET_KEY", "dev"))
    app.register_blueprint(homes, url_prefix="/homes")

    @app.get("/health")
    def health():
        return {"status": "ok"}

    return app

def test_health():
    client = create_app().test_client()
    response = client.get("/health")
    assert response.status_code == 200
    assert response.get_json()["status"] == "ok"
    return response.status_code
```

Result: `test_health()` returns `200`. `response.get_json()` is `{"status": "ok"}`. No port is opened.

Why: each call to `create_app()` builds a fresh app, so a test can pass different config. `os.environ.get("SECRET_KEY", "dev")` uses the environment when it is set, and the local fallback otherwise. A deployed process should set `SECRET_KEY`.

**Case: a missing id stays a 404 inside the blueprint**

```python
from flask import abort

@homes.get("/<home_id>")
def one_home(home_id):
    row = find_home(home_id)
    if row is None:
        abort(404)
    return row
```

Result: with the blueprint mounted at `/homes`, `GET /homes/missing` returns status 404 when the lookup returns `None`. The 404 handler from the previous case supplies `{"error": "not found"}`.

Why: `abort(404)` raises, and the app error handler runs. Returning nothing would be status 200.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `url_for("list_homes")` after a blueprint | The endpoint is not found | `url_for("homes.list_homes")` |
| A handler that returns 200 | The client thinks the missing page was fine | Return the error status again: `return {...}, 404` |
| `debug=True` in the factory used in production | The debugger can execute code | Keep `debug` off unless the process is local |
| Sharing one global app across tests | Config and routes leak between tests | Build a new app in `create_app()` for each test |

### Tip

The endpoint string is `blueprint.function` once you register a blueprint. Search for that string when `url_for` cannot find a view.

### Practice

Mount a `homes` blueprint at `/homes` with `GET /` returning an empty JSON list. Add a 404 handler that returns `{"error": "not found"}` with status 404.

### Solution

```python
from flask import Blueprint, Flask

homes = Blueprint("homes", __name__)

@homes.get("/")
def list_homes():
    return []

app = Flask(__name__)
app.register_blueprint(homes, url_prefix="/homes")

@app.errorhandler(404)
def missing(_error):
    return {"error": "not found"}, 404
```

Result: `GET /homes/` is status 200 and `[]`. Any unknown path is status 404 and `{"error": "not found"}`. `url_for("homes.list_homes")` is `/homes/`.

## Drill

1. Write `GET /health` that returns JSON.
2. Write `GET /homes/<home_id>` that calls `abort(404)` when your lookup returns `None`.
3. Name the two places a `city` field can arrive: the query string and the JSON body.

### Solution 1

```python
@app.get("/health")
def health():
    return {"status": "ok"}
```

Result: `GET /health` returns status 200 and `{"status": "ok"}`.

### Solution 2

```python
from flask import abort

@app.get("/homes/<home_id>")
def one_home(home_id):
    row = find_home(home_id)
    if row is None:
        abort(404)
    return row
```

Result: a lookup of `None` returns status 404. A found row is JSON with status 200.

### Solution 3

The query string is `request.args`, as in `GET /homes?city=Irvine` and `request.args.get("city")`.

The JSON body is `request.json`, or `request.get_json(silent=True)`, as in a `POST` with `Content-Type: application/json` and body `{"city": "Irvine"}`.

Result: those two calls read different parts of the request. A form body is a third place, `request.form`, and it is not one of the two the question names.
