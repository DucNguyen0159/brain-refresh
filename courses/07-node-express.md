# Node.js and Express

Course 7 of 15. Previous: [06-nextjs.md](06-nextjs.md). Next: [08-python.md](08-python.md).

Node.js runs JavaScript outside the browser. Express is a library that maps an HTTP method and a path to a function. The first half of this course is Node. The second half is Express.

## Contents

1. [The process](#1-the-process)
2. [The event loop](#2-the-event-loop)
3. [Modules, packages, and npm](#3-modules-packages-and-npm)
4. [Paths, files, and environment](#4-paths-files-and-environment)
5. [Events and errors](#5-events-and-errors)
6. [HTTP without a framework](#6-http-without-a-framework)
7. [Express routes](#7-express-routes)
8. [Middleware, routers, and errors](#8-middleware-routers-and-errors)

## 1. The process

`node app.js` starts a process, runs the file, and exits when the work is done and nothing is still scheduled. A server stays alive because it is waiting for connections.

| Object | Meaning |
| --- | --- |
| `process.argv` | Command-line words. Index 0 is `node`, index 1 is the script |
| `process.env` | Environment variables, all strings |
| `process.cwd()` | The folder you launched from |
| `process.exitCode = 1` | Tell the shell the run failed, after the script finishes |
| `process.stdin` / `stdout` / `stderr` | The three standard streams |

```javascript
const args = process.argv.slice(2);
console.log(args.join(" "));
```

Node is single-threaded for your JavaScript. Section 2 is what that means for a request.

## 2. The event loop

Your JavaScript runs on one thread. The loop runs that code until the function returns, then it runs timers that are due, then it runs finished file and network callbacks. A second request cannot run while a function is still in the middle of a loop.

```javascript
import { createReadStream } from "node:fs";
import { readFile } from "node:fs/promises";

const text = await readFile("notes.txt", "utf8");

const stream = createReadStream("notes.txt", "utf8");
stream.on("data", (chunk) => {
  console.log(chunk.length);
});
```

| Call | Your function | The rest of the server |
| --- | --- | --- |
| `readFile` | Starts the read and waits with `await`. The function is paused, not spinning | Other requests can run while the disk works. The whole file arrives as one string |
| `createReadStream` | Starts the read and returns. Chunks arrive later on `"data"` | Same background read, in pieces. A large file does not become one giant string |
| A long `for` loop | Stays on the thread until the loop ends | No other request, timer, or file callback runs until it finishes |

`await` does not mean the thread is asleep. It means this function stepped aside so the loop can run other work. A tight loop never steps aside.

## 3. Modules, packages, and npm

Each file is a module. Other files see only its exports.

```javascript
export function add(a, b) {
  return a + b;
}
```

```javascript
import { add } from "./add.js";
```

| System | Looks like | When |
| --- | --- | --- |
| ES modules | `import` / `export` | `"type": "module"` in `package.json`, or a `.mjs` file |
| CommonJS | `require` and `module.exports` | The older default. You will read it in older tutorials |

`npm install express` downloads a package into `node_modules` and records it in `package.json`. `npm install` with no name installs whatever `package.json` lists.

| Field or file | Role |
| --- | --- |
| `dependencies` | Needed to run |
| `devDependencies` | Needed to develop, such as a test runner |
| `scripts` | `npm test` runs the `test` script |
| `package-lock.json` | The exact versions that were installed. Commit it |
| `node_modules` | The downloaded files. Do not commit it |

Never commit tokens. Read them from the environment.

## 4. Paths, files, and environment

```javascript
import { readFile, writeFile } from "node:fs/promises";
import path from "node:path";
import { fileURLToPath } from "node:url";

const here = path.dirname(fileURLToPath(import.meta.url));
const file = path.join(here, "notes.txt");
const text = await readFile(file, "utf8");
await writeFile(file, text + "\n", "utf8");
```

| Tool | Use |
| --- | --- |
| `path.join` | Build a path with the right separators |
| `path.resolve` | An absolute path |
| `import.meta.url` | The URL of this module. Convert it to a path before you join |
| `fs/promises` | Async file functions that return promises |
| `fs` sync functions | `readFileSync`. Fine for a short script. They block a server |

| Call | What it does | Sync twin |
| --- | --- | --- |
| `readFile(path, "utf8")` | The whole file as a string | `readFileSync` |
| `writeFile(path, text, "utf8")` | Replace the file. Creates it if missing | `writeFileSync` |
| `appendFile(path, text, "utf8")` | Add to the end | `appendFileSync` |
| `mkdir(path, { recursive: true })` | Create a directory, and parents if asked | `mkdirSync` |
| `readdir(path)` | The names inside a directory | `readdirSync` |
| `stat(path)` | Size, times, and whether it is a file | `statSync` |
| `access(path)` | Resolves if you may use it. Rejects if not | `accessSync` |
| `rm(path, { recursive: true })` | Delete a file or a tree | `rmSync` |
| `rename(from, to)` | Move or rename | `renameSync` |

`path.join(here, "..", "data", "a.csv")` climbs one directory. Do not concatenate user input into a path and read it. A value such as `../../.env` can escape the folder you intended. Resolve the path and check that it still starts with the allowed root.

| Environment | Detail |
| --- | --- |
| `process.env.PORT` | A string or `undefined` |
| Missing variable | Check it at startup and exit with a clear message |
| `.env` files | A loader such as `dotenv` puts them into `process.env`. The file stays out of git |

## 5. Events and errors

Node's `EventEmitter` is a list of listeners.

```javascript
import { EventEmitter } from "node:events";

const bus = new EventEmitter();
bus.on("ready", (name) => {
  console.log(name);
});
bus.emit("ready", "api");
```

An error thrown in a promise, with nobody awaiting it, can crash the process. `await` it or attach `.catch`. An `error` event on a stream, with no listener, can crash the process too.

| Failure | What you do |
| --- | --- |
| Expected, such as a missing file | `try/catch` and a useful message |
| Programmer mistake | Let it throw during development |
| Request handler | Catch it and send a 500. Do not leak stack traces to the client in production |

## 6. HTTP without a framework

```javascript
import http from "node:http";

const server = http.createServer((req, res) => {
  if (req.method === "GET" && req.url === "/health") {
    res.writeHead(200, { "content-type": "application/json" });
    res.end(JSON.stringify({ status: "ok" }));
    return;
  }
  res.writeHead(404, { "content-type": "application/json" });
  res.end(JSON.stringify({ error: "not found" }));
});

server.listen(3000);
```

| Piece | Meaning |
| --- | --- |
| `req.method` | `GET`, `POST`, and the rest |
| `req.url` | Path plus query string |
| `res.writeHead` | Status and headers. Must happen before the body |
| `res.end` | Finish the response |
| Status | 200 ok, 201 created, 400 bad input, 401 unauthenticated, 403 forbidden, 404 missing, 500 server fault |

Express exists so you do not parse URLs and bodies by hand. Know this layer so Express is not magic.

## 7. Express routes

```javascript
import express from "express";

const app = express();
app.use(express.json());

app.get("/health", (_req, res) => {
  res.json({ status: "ok" });
});

app.get("/homes/:id", (req, res) => {
  res.json({ id: req.params.id });
});

app.post("/homes", (req, res) => {
  const { city } = req.body ?? {};
  if (typeof city !== "string" || city.trim() === "") {
    res.status(400).json({ error: "city is required" });
    return;
  }
  res.status(201).json({ city: city.trim() });
});

app.listen(3000);
```

| Source | Example | Lives in |
| --- | --- | --- |
| Route param | `/homes/:id` | `req.params.id` |
| Query string | `/homes?city=Irvine` | `req.query.city` |
| JSON body | `{ "city": "Irvine" }` | `req.body`, after `express.json()` |
| Header | `Authorization` | `req.get("authorization")` |

| Method helper | Typical use |
| --- | --- |
| `app.get` | Read |
| `app.post` | Create |
| `app.put` | Replace |
| `app.patch` | Partial update |
| `app.delete` | Remove |

`res.json` sets the JSON content type and ends the response. `res.status(404).json(...)` sets the status first. Calling `res.json` twice throws. `return` after you send an error.

The same request, in the two APIs:

| Job | `node:http` | Express |
| --- | --- | --- |
| Method | `req.method` | `req.method`, and you register `app.get` or `app.post` |
| Path | `req.url` includes the query string. You split it | `req.path` is the path. `req.query` is parsed |
| Route param | You match the path yourself | `req.params.id` from `/homes/:id` |
| Body | Read the stream and `JSON.parse` it | `req.body`, after `express.json()` |
| Header | `req.headers["content-type"]` | `req.get("content-type")` |
| Status and body | `res.writeHead(status, headers)` then `res.end(body)` | `res.status(status).json(body)` |
| Send a file | Set headers and stream the file | `res.sendFile(path)` |

`express.urlencoded` parses HTML form bodies. `express.static("public")` serves files.

## 8. Middleware, routers, and errors

Middleware is a function of `(req, res, next)`. It runs in the order you `use` it. It either sends a response or calls `next()`.

```javascript
function requireJson(req, res, next) {
  if (req.method === "POST" && !req.is("application/json")) {
    res.status(415).json({ error: "send JSON" });
    return;
  }
  next();
}

app.use(requireJson);
```

| Middleware job | Example |
| --- | --- |
| Parse the body | `express.json()` |
| Log | Print method and URL, then `next()` |
| Authenticate | Read a session. `401` if there is none. `next()` if there is |
| Authorize | `403` when the role is wrong |
| Serve files | `express.static` |
| Handle errors | Four arguments: `(error, req, res, next)` |

An async handler's rejection does not reach the error middleware unless you pass it on:

```javascript
function asyncHandler(fn) {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

app.get("/homes", asyncHandler(async (_req, res) => {
  const rows = await loadHomes();
  res.json(rows);
}));

app.use((error, _req, res, _next) => {
  console.error(error);
  res.status(500).json({ error: "failed" });
});
```

A router is a mini app you mount on a path.

```javascript
const homes = express.Router();
homes.get("/", (_req, res) => res.json([]));
app.use("/homes", homes);
```

| Practice | Why |
| --- | --- |
| Validate input on the server | The client can send anything |
| Do not reflect stack traces | They describe your files |
| Keep secrets in the environment | A committed key is public |
| Limit body size | `express.json({ limit: "100kb" })` |
| CORS | Browsers block other origins unless you send the headers. A server-to-server call is not subject to CORS |

## Drill

1. Read `process.argv` and print every word after the script name.
2. Write `GET /health` and `POST /notes` that returns 400 when `text` is missing.
3. Say what `next` is for, in one sentence.
4. Say why a long `for` loop in a request handler blocks other requests, and `await readFile` does not.
