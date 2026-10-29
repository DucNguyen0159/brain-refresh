# Flask

Course 9 of 15. Previous: [08-python.md](08-python.md). Next: [10-sql.md](10-sql.md).

Flask maps a URL and an HTTP method to a Python function. You install it with `pip install flask`. A small app is one file. A larger app uses blueprints and a factory.

## Contents

1. [An application and a route](#1-an-application-and-a-route)
2. [Request data](#2-request-data)
3. [Responses and status codes](#3-responses-and-status-codes)
4. [Templates](#4-templates)
5. [Static files, config, and sessions](#5-static-files-config-and-sessions)
6. [Blueprints and errors](#6-blueprints-and-errors)

## 1. An application and a route

```python
from flask import Flask

app = Flask(__name__)

@app.get("/health")
def health():
    return {"status": "ok"}

if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

| Piece | Meaning |
| --- | --- |
| `Flask(__name__)` | The application. `__name__` helps Flask find templates and static files |
| `@app.get("/health")` | Register this function for `GET /health` |
| `@app.route("/homes", methods=["GET", "POST"])` | Several methods on one path |
| `debug=True` | Reloads on save and shows a debugger. Development only |
| Return a dict | Flask turns it into a JSON response |

A variable in the path is a function argument:

```python
@app.get("/homes/<home_id>")
def home(home_id):
    return {"id": home_id}

@app.get("/reports/<int:year>")
def report(year):
    return {"year": year}
```

| Converter | Accepts |
| --- | --- |
| default `string` | Text without a slash |
| `int` | An integer. A non-integer does not match the route |
| `float` | A float |
| `path` | Text that may contain slashes |
| `uuid` | A UUID |

## 2. Request data

```python
from flask import request

@app.get("/homes")
def homes():
    city = request.args.get("city", "")
    return {"city": city}

@app.post("/homes")
def create_home():
    data = request.get_json(silent=True) or {}
    city = data.get("city", "")
    if not isinstance(city, str) or not city.strip():
        return {"error": "city is required"}, 400
    return {"city": city.strip()}, 201
```

| Source | API |
| --- | --- |
| Query string `?city=Irvine` | `request.args.get("city")` |
| Form body | `request.form.get("city")` |
| JSON body | `request.get_json()` |
| Header | `request.headers.get("Authorization")` |
| Uploaded file | `request.files["photo"]` |
| Path converter | The function argument |

`get` returns `None` when the key is missing, or the default you pass. Indexing `request.args["city"]` raises when the key is missing.

| | `request.args` | `request.form` | `request.get_json()` |
| --- | --- | --- | --- |
| Where the data is | The query string, `?city=Irvine` | A form body | A JSON body |
| Typical method | Any, often `GET` | `POST` | `POST` or `PUT` |
| Content type | None. It is in the URL | `application/x-www-form-urlencoded` or `multipart/form-data` | `application/json` |
| One value | `.get("city")` | `.get("city")` | The dict's `.get("city")` |
| Several values for one key | `.getlist("city")` | `.getlist("city")` | A JSON list under that key |
| Missing key | `.get` returns `None`. `["city"]` raises | Same | `.get` returns `None`. A missing body returns `None` when `silent=True` |
| A file | No | `request.files["photo"]` when the form is multipart | No |

| Return | Status | Body | Headers you set |
| --- | --- | --- | --- |
| `"hello"` | 200 | Text | Flask picks the content type |
| `{"ok": True}` | 200 | JSON | Flask picks JSON |
| `(body, 201)` | The number you gave | `body` | Flask picks from the body type |
| `(body, 201, {"X-Request": "1"})` | 201 | `body` | Those, plus what Flask adds |
| `jsonify(obj)` | 200 unless you change the response | JSON | You can set more on the returned response |
| `abort(404)` | 404 | The error handler's body | The error handler's |

Validate on the server. The client can send any body.

## 3. Responses and status codes

| Return | What Flask sends |
| --- | --- |
| `"hello"` | HTML or text, status 200 |
| `{"ok": True}` | JSON, status 200 |
| `({"error": "missing"}, 400)` | JSON and that status |
| `jsonify(obj)` | JSON, and you can keep building the response |
| `redirect(url)` | A redirect response |
| `send_file(path)` | A file download or inline file |
| `make_response` | A response object you can set headers on |

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

```python
from flask import abort

@app.get("/homes/<home_id>")
def one_home(home_id):
    row = find_home(home_id)
    if row is None:
        abort(404)
    return row
```

`abort` raises an HTTP exception. An error handler can turn it into a page or JSON.

## 4. Templates

Flask looks for HTML files in a `templates` folder next to the app. The language inside them is Jinja.

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

| Jinja | Meaning |
| --- | --- |
| `{{ name }}` | Print the value, escaped. A `<` in the value stays text |
| `{% if n %}...{% endif %}` | A branch |
| `{% for item in items %}...{% endfor %}` | A loop |
| `{{ value \| safe }}` | Do not escape. Only for HTML you generated yourself |
| `{% extends "base.html" %}` | This file fills blocks in a parent |
| `{% block body %}{% endblock %}` | A hole the child fills |
| `url_for("index")` | Build a URL from a function name, so routes can change |

`url_for("home", home_id="R1")` fills a path variable. `url_for("static", filename="app.css")` points at the static folder.

Autoescaping is why you put user text in `{{ }}` and not in `| safe`.

## 5. Static files, config, and sessions

| Folder | Served as |
| --- | --- |
| `static/app.css` | `/static/app.css` |
| `templates/index.html` | Not served directly. `render_template` reads it |

```python
import os

app.config["SECRET_KEY"] = os.environ["SECRET_KEY"]
app.config["MAX_CONTENT_LENGTH"] = 1024 * 1024
```

`SECRET_KEY` signs the session cookie. Without it, sessions refuse to work. Read it from the environment. Do not commit it.

```python
from flask import session

@app.post("/city")
def save_city():
    session["city"] = request.form.get("city", "")
    return redirect(url_for("index"))
```

| Store | Lifetime |
| --- | --- |
| `session` | A signed cookie. Small. Survives requests from that browser |
| `g` | Data for the current request only |
| A global variable | Shared by every request in the process. Easy to leak one user's data to another |

`MAX_CONTENT_LENGTH` rejects huge uploads.

The development server is `flask run` or `app.run`. It is not the server you use under load. A production process uses a WSGI server and points it at `app`.

## 6. Blueprints and errors

A blueprint is a group of routes you can mount.

```python
from flask import Blueprint

homes = Blueprint("homes", __name__)

@homes.get("/")
def list_homes():
    return []

app.register_blueprint(homes, url_prefix="/homes")
```

`url_for("homes.list_homes")` uses the blueprint name and the function name.

```python
@app.errorhandler(404)
def missing(_error):
    return {"error": "not found"}, 404

@app.errorhandler(500)
def failed(_error):
    return {"error": "failed"}, 500
```

| Practice | Why |
| --- | --- |
| Check types and presence of JSON fields | `get_json` may be `None` |
| Return 404 for a missing id | An empty 200 looks like success |
| Keep `debug` off in production | The debugger can execute code |
| Log the exception, send a short message | The stack is for you |
| One app factory `create_app()` | Tests can build an app with different config |

```python
def create_app():
    app = Flask(__name__)
    app.config.from_mapping(SECRET_KEY=os.environ.get("SECRET_KEY", "dev"))
    app.register_blueprint(homes, url_prefix="/homes")
    return app
```

A test client calls routes without opening a port:

```python
def test_health():
    client = create_app().test_client()
    response = client.get("/health")
    assert response.status_code == 200
    assert response.get_json()["status"] == "ok"
```

## Drill

1. Write `GET /health` that returns JSON.
2. Write `GET /homes/<home_id>` that calls `abort(404)` when your lookup returns `None`.
3. Name the two places a `city` field can arrive: the query string and the JSON body.
