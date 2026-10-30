# Node.js and Express

Course 7 of 15. Previous: [06-nextjs.md](06-nextjs.md). Next: [08-python.md](08-python.md).

Node.js runs JavaScript outside the browser. Express is a library that maps an HTTP method and a path to a function. The first half of this course is Node. The second half is Express. Every section has the same parts: the concept, the terms, the methods, several cases with what prints or what the server does, common mistakes, a tip, and a practice with a solution.

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

### Concept

`node app.js` starts a process, runs the file, and exits when the work is done and nothing is still scheduled. A server stays alive because it is waiting for connections. Node is single-threaded for your JavaScript. Section 2 is what that means for a request. `process.argv` is the command-line words. Index 0 is `node`, index 1 is the script, and the rest are the words you passed. `process.env` values are all strings. `process.exitCode = 1` tells the shell the run failed after the script finishes, instead of stopping in the middle.

### Terms

| Term | Meaning |
| --- | --- |
| Process | One run of `node`. It has an id, an environment, and three standard streams |
| `process.argv` | Command-line words. Index 0 is `node`, index 1 is the script |
| `process.env` | Environment variables. Every value is a string, or the name is missing |
| `process.cwd()` | The folder you launched from, which may differ from the folder that holds the file |
| Exit code | `0` means success. `process.exitCode = 1` means failure, reported when the process actually ends |

### Methods

| Object | Meaning |
| --- | --- |
| `process.argv` | Command-line words. Index 0 is `node`, index 1 is the script |
| `process.argv.slice(2)` | The words after the script name |
| `process.env` | Environment variables, all strings |
| `process.cwd()` | The folder you launched from |
| `process.exitCode = 1` | Tell the shell the run failed, after the script finishes |
| `process.stdin` / `stdout` / `stderr` | The three standard streams. `console.log` writes to stdout. `console.error` writes to stderr |

### Examples

**Case: words after the script name**

```javascript
const args = process.argv.slice(2);
console.log(args.join(" "));
```

Result: `node app.js one two` prints `one two`. `node app.js` prints a blank line.

Why: `process.argv[0]` is the `node` binary and `process.argv[1]` is `app.js`. `slice(2)` drops both. The remaining words are strings, even when they look like numbers.

**Case: the environment is strings**

```javascript
const port = process.env.PORT;
console.log(typeof port);
console.log(port);
```

Result: with `PORT=3000`, this prints `string` and then `3000`. With `PORT` unset, it prints `undefined` and then `undefined`.

Why: `process.env` stores strings. `"3000" + 1` is `"30001"`, not `3001`. Convert with `Number(port)` after you have checked that the name exists.

**Case: the launch folder is not the file folder**

```javascript
console.log(process.cwd());
```

Result: prints the folder you ran `node` from. Running it from `/tmp` prints `/tmp` even when `app.js` lives somewhere else.

Why: `process.cwd()` is the launch folder. A relative path such as `notes.txt` is resolved from there, not from the module, unless you build the path from the module URL yourself. Section 4 does that.

**Case: a failing run still finishes**

```javascript
const args = process.argv.slice(2);
if (args.length === 0) {
  console.error("missing name");
  process.exitCode = 1;
} else {
  console.log(args.join(" "));
}
```

Result: `node app.js` prints `missing name` on stderr and the shell sees exit code 1 after the script finishes. `node app.js Ada` prints `Ada` and exits 0. A listening server does not reach this exit, because it is still waiting for connections.

Why: `exitCode` records the status and lets scheduled work finish. `process.exit(1)` stops immediately, which can cut off a pending write. Prefer `exitCode` unless you must stop now. `console.error` uses stderr so a pipe of stdout stays clean.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Treating `argv[0]` as the first user word | You print `node`, or you drop a real argument | `process.argv.slice(2)` |
| Assuming `process.env.PORT` is a number | Arithmetic concatenates strings | Check for `undefined`, then `Number(port)` |
| `process.exit(1)` in the middle of a request | Pending writes and open handles stop at once | `process.exitCode = 1` and return |
| A relative file path that depends on the shell folder | The script cannot find `notes.txt` when launched from another directory | Build the path from the module, as in section 4 |

### Tip

The process ends when the call stack is empty and nothing is still scheduled. A server stays up only because the open port counts as scheduled work.

### Practice

Read `process.argv` and print every word after the script name.

### Solution

```javascript
const args = process.argv.slice(2);
console.log(args.join(" "));
```

Result: `node app.js one two` prints `one two`. Index 0 and index 1 are `node` and the script, so they are not in the output.

## 2. The event loop

### Concept

Your JavaScript runs on one call stack. The event loop takes the next task only when that stack is empty. Microtasks run first: promise reactions (`then`, `catch`, `finally`), `queueMicrotask`, and the continuation after an `await`. Macrotasks run after the microtasks queued so far. Timers (`setTimeout`, `setInterval`), I/O callbacks, and `setImmediate` are macrotasks. `process.nextTick` is its own queue. In a CommonJS file it drains before promise microtasks. In an ES module the file itself is already running as a microtask, so a promise queued there can run before `nextTick`. `await` does not mean the thread is asleep. It means this function stepped aside so the loop can run other work. A tight loop never steps aside. A second request cannot run while a function is still in the middle of a loop.

### Terms

| Term | Meaning |
| --- | --- |
| Call stack | The synchronous functions running now. Nothing else runs until they return |
| Microtask | A promise callback or `queueMicrotask`. Drained when the stack empties, before the next macrotask |
| Macrotask | A timer, a finished file or network callback, or `setImmediate` |
| `process.nextTick` | A queue that runs before promise microtasks in CommonJS, and can run after them when the ES module body is already inside a microtask |
| Turn | One trip through ready queues. A long synchronous loop is still the same turn |

### Methods

| Call | Your function | The rest of the server |
| --- | --- | --- |
| `readFile` | Starts the read and waits with `await`. The function is paused, not spinning | Other requests can run while the disk works. The whole file arrives as one string |
| `createReadStream` | Starts the read and returns. Chunks arrive later on `"data"` | Same background read, in pieces. A large file does not become one giant string |
| A long `for` loop | Stays on the thread until the loop ends | No other request, timer, or file callback runs until it finishes |
| `setTimeout(fn, 0)` | Queues a macrotask | Runs after the stack clears and after microtasks |
| `Promise.resolve().then(fn)` | Queues a microtask | Runs after the stack clears, before timer macrotasks |
| `process.nextTick(fn)` | Queues the next-tick queue | In CommonJS, before promise microtasks. Do not use it to starve I/O |

### Examples

**Case: the stack finishes, then a microtask, then a macrotask**

```javascript
console.log("start");
setTimeout(() => console.log("macrotask"), 0);
Promise.resolve().then(() => console.log("microtask"));
console.log("end");
```

Result: prints `start`, then `end`, then `microtask`, then `macrotask`.

Why: `start` and `end` run on the call stack. When that stack is empty, promise microtasks run. The timer is a macrotask, so it waits until those microtasks finish. A delay of `0` still does not jump the queue.

**Case: `nextTick` and promises in CommonJS**

```javascript
const { nextTick } = require("node:process");

console.log("stack");
nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("microtask"));
setTimeout(() => console.log("macrotask"), 0);
```

Result: prints `stack`, then `nextTick`, then `microtask`, then `macrotask`.

Why: a CommonJS file runs as a plain call. After it returns, Node drains the next-tick queue and then promise microtasks, and only then the timer macrotask. An ES module is loaded inside the microtask queue, so the same three callbacks can print the promise before `nextTick`. Learn the two queues. Do not memorize one `nextTick` order for both module systems.

**Case: `await readFile` steps aside**

```javascript
import { readFile } from "node:fs/promises";

console.log("before");
const text = await readFile("notes.txt", "utf8");
console.log(text.length);
console.log("after");
```

Result: prints `before`, waits while the disk reads, then prints the character count and `after`. Other request callbacks can run during that wait. The whole file arrives as one string.

Why: `await` pauses this function and returns to the event loop. The thread is not asleep. A second request can run on the call stack until this read finishes and the rest of this function is queued as a microtask. A tight loop never steps aside, so it does not give that chance.

**Case: a long loop blocks the process**

```javascript
console.log("start");
const begin = Date.now();
while (Date.now() - begin < 50) {
  // stay on the call stack
}
console.log("end");
```

```javascript
import { createReadStream } from "node:fs";

const stream = createReadStream("notes.txt", "utf8");
stream.on("data", (chunk) => {
  console.log(chunk.length);
});
```

Result: the loop prints `start`, then `end` about 50 milliseconds later. No timer, microtask, or other request runs between those two lines. The stream prints one length per chunk as `"data"` events arrive, and it does not build one giant string.

Why: the `while` loop stays on the call stack, so the event loop cannot take a macrotask or a microtask. `createReadStream` returns immediately and delivers pieces later. `readFile` is the opposite shape: one string, after one wait. Use the stream when the file is large. Use `await readFile` when you need the whole small file and you can step aside.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A long `for` or `while` inside a request | No other request, timer, or file callback runs until the loop ends | Move the work off the hot path, or break it up so the stack can clear |
| Expecting `setTimeout(fn, 0)` to run before a promise | The timer is a macrotask and loses to microtasks | Put the follow-up work in the promise chain if it must stay ahead of timers |
| Assuming `await` freezes the thread | Other callbacks run while this function is paused | Reserve that pause for I/O. Do not busy-wait |
| One `nextTick` order copied from CommonJS into an ES module | The promise can run first, because the module body is already a microtask | Treat `nextTick` and promises as two queues |

### Tip

If a handler must do heavy synchronous work, every other connection waits. `await` on I/O is the case that lets them through.

### Practice

Say why a long `for` loop in a request handler blocks other requests, and `await readFile` does not.

### Solution

The loop stays on the call stack until it finishes, so the event loop cannot start another request, a timer macrotask, or a file callback. `await readFile` pauses that function. The thread is free, and other requests run while the disk works. The file then arrives as one string.

```javascript
import { readFile } from "node:fs/promises";

const text = await readFile("notes.txt", "utf8");
console.log(text.length);
```

Result: other requests can run during the `await`. They cannot run during a tight loop that never returns.

## 3. Modules, packages, and npm

### Concept

Each file is a module. Other files see only its exports. ES modules use `import` and `export`. They are on when `package.json` says `"type": "module"`, or when the file ends in `.mjs`. CommonJS uses `require` and `module.exports`. It is the older default, and you will read it in older tutorials. A `.cjs` file stays CommonJS even in a `"type": "module"` package. `npm install express` downloads a package into `node_modules` and records it in `package.json`. `npm install` with no name installs whatever `package.json` lists. Never commit tokens. Read them from the environment.

### Terms

| Term | Meaning |
| --- | --- |
| ES module | `import` / `export`. `"type": "module"` or a `.mjs` file. Relative imports include the extension |
| CommonJS | `require` and `module.exports`. The older default for a `.js` file |
| `dependencies` | Needed to run |
| `devDependencies` | Needed to develop, such as a test runner |
| `package-lock.json` | The exact versions that were installed. Commit it |
| `node_modules` | The downloaded files. Do not commit it |

### Methods

| Field, file, or command | Role |
| --- | --- |
| `export` / `import` | Publish a name, or read one, in an ES module |
| `module.exports` / `require` | The CommonJS pair |
| `npm install express` | Download Express into `node_modules` and record it in `dependencies` |
| `npm install` | With no package name, install whatever `package.json` lists |
| `npm install -D vitest` | Record a dev dependency |
| `scripts` | `npm test` runs the `test` script |
| `package-lock.json` | Lock the exact versions. Commit it |
| `node_modules` | Installed files. Do not commit them |

### Examples

**Case: an ES module exports a function**

```javascript
export function add(a, b) {
  return a + b;
}
```

```javascript
import { add } from "./add.js";

console.log(add(2, 3));
```

Result: prints `5`.

Why: other files see only the export. The import path includes `.js` because Node's ES module loader does not guess the extension. This file is ESM when the package has `"type": "module"` or the name ends in `.mjs`.

**Case: CommonJS uses `require`**

```javascript
function add(a, b) {
  return a + b;
}

module.exports = { add };
```

```javascript
const { add } = require("./add.js");

console.log(add(2, 3));
```

Result: prints `5`.

Why: CommonJS is the older system. `module.exports` is the public object, and `require` reads it. A `.js` file uses this when `package.json` has no `"type": "module"`. A `.cjs` file is CommonJS even when the package is a module. `require` is not defined in an ES module file.

**Case: `npm install` writes two different records**

```text
npm install express
npm install
```

Result: `npm install express` downloads Express into `node_modules` and adds it under `dependencies` in `package.json`. `npm install` with no name installs the packages already listed. `package-lock.json` records the exact versions.

Why: the manifest is what a teammate installs. The lockfile makes that install the same versions. Commit `package-lock.json`. Do not commit `node_modules`. `devDependencies` are for tools such as a test runner. `npm test` runs the `test` script in `scripts`.

**Case: a token stays in the environment**

```javascript
const token = process.env.API_TOKEN;
if (token === undefined) {
  console.error("API_TOKEN is missing");
  process.exitCode = 1;
} else {
  console.log(token.length);
}
```

Result: when `API_TOKEN` is unset, the process prints `API_TOKEN is missing` and finishes with exit code 1. When it is set, the process prints the length, not a reason to commit the value.

Why: a token written into a source file or into git is public to everyone who can read the repo. Read it from the environment. `dependencies` do not hold tokens. The length print is a stand-in so the value itself is not logged.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `import { add } from "./add"` in Node ESM | `ERR_MODULE_NOT_FOUND`. The loader does not add `.js` | Import `./add.js` |
| `require` inside an ES module | `ReferenceError: require is not defined` | `import`, or rename the file to `.cjs` if it must stay CommonJS |
| Committing `node_modules` | The repo copies every installed file, and versions drift from the lockfile | Commit `package-lock.json` and install locally |
| Pasting a token into `package.json` or source | Anyone with the repo can read it | `process.env`, and leave the value out of git |

### Tip

If the file uses `import` and Node says it cannot use an import statement, the package is still CommonJS. Add `"type": "module"` or use `.mjs`.

### Practice

Export `add` from an ES module and print `add(2, 3)` from another file.

### Solution

```javascript
export function add(a, b) {
  return a + b;
}
```

```javascript
import { add } from "./add.js";

console.log(add(2, 3));
```

Result: prints `5`. The second file can see `add` only because the first file exported it.

## 4. Paths, files, and environment

### Concept

Build paths with `path.join` and `path.resolve`, not with string concatenation. In an ES module, `__dirname` does not exist. `import.meta.url` is the URL of this module. Convert it with `fileURLToPath` before you join. Current Node also provides `import.meta.dirname`. `fs/promises` functions return promises and let the event loop run. The sync twins, such as `readFileSync`, block a server. `process.env.PORT` is a string or `undefined`. Check a missing variable at startup and exit with a clear message. A loader such as `dotenv` puts a `.env` file into `process.env`. The file stays out of git.

### Terms

| Term | Meaning |
| --- | --- |
| `path.join` | Build a path with the right separators |
| `path.resolve` | An absolute path, collapsing `..` |
| `import.meta.url` | The URL of this module. Convert it to a path before you join |
| `fs/promises` | Async file functions that return promises |
| Sync fs | `readFileSync` and the other `Sync` twins. Fine for a short script. They block a server |
| `process.env.PORT` | A string or `undefined` |

### Methods

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

### Examples

**Case: a path next to the module**

```javascript
import { readFile, writeFile } from "node:fs/promises";
import path from "node:path";
import { fileURLToPath } from "node:url";

const here = path.dirname(fileURLToPath(import.meta.url));
const file = path.join(here, "notes.txt");
const text = await readFile(file, "utf8");
await writeFile(file, text + "\n", "utf8");
```

Result: reads `notes.txt` beside this module, then writes that text back with one extra newline. `writeFile` creates the file if it is missing. `appendFile` would have added to the end instead of replacing it.

Why: `import.meta.url` is a `file:` URL. `fileURLToPath` makes a filesystem path, and `path.dirname` is the folder. `path.join` inserts the separator for the operating system. `import.meta.dirname` is the same folder on current Node. `process.cwd()` would have used the launch folder instead.

**Case: `..` climbs, and user input must not**

```javascript
import path from "node:path";

const here = "/srv/app";
const notes = path.join(here, "..", "data", "a.csv");
const root = path.resolve("/data/notes");
const target = path.resolve(root, "../../.env");
const inside = target === root || target.startsWith(root + path.sep);
console.log(notes);
console.log(inside);
```

Result: `notes` is `/srv/data/a.csv` on Unix, one directory above `app`. `inside` is `false`, because `../../.env` resolved outside `/data/notes`.

Why: `path.join(here, "..", "data", "a.csv")` climbs one directory on purpose. Do not concatenate user input into a path and read it. A value such as `../../.env` can escape the folder you intended. Resolve the path and check that it still equals the allowed root or starts with that root plus `path.sep`. A bare `startsWith(root)` also matches a sibling folder whose name only begins with the root.

**Case: a sync read blocks**

```javascript
import { readFileSync } from "node:fs";

const text = readFileSync("notes.txt", "utf8");
console.log(text.length);
```

Result: prints the character count. While the disk works, no other request callback runs.

Why: `readFileSync` stays on the call stack. That is fine for a short script. In a server, use `readFile` from `node:fs/promises` so the function can `await`. The promise version still returns the whole file as one string. The sync twins of the other calls (`writeFileSync`, `mkdirSync`, `readdirSync`, `statSync`, `accessSync`, `rmSync`, `renameSync`, `appendFileSync`) block in the same way.

**Case: a missing port stops startup**

```javascript
const port = process.env.PORT;
if (port === undefined || port.trim() === "") {
  console.error("PORT is missing");
  process.exitCode = 1;
} else {
  console.log(port);
}
```

Result: with `PORT` unset, prints `PORT is missing` and finishes with exit code 1. With `PORT=3000`, prints `3000`.

Why: a missing variable is `undefined`, and a present one is a string. Check it at startup and exit with a clear message, before you call `listen`. A `.env` file is not loaded by Node itself. A loader such as `dotenv` puts those names into `process.env`. The file stays out of git.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `__dirname` in an ES module | `ReferenceError`. That name is CommonJS | `fileURLToPath(import.meta.url)` or `import.meta.dirname` |
| `path` built by concatenating user input | `../../.env` can escape the folder | `path.resolve` and a root check that includes `path.sep` |
| `readFileSync` on a request | The process handles no other request until the disk returns | `await readFile` from `node:fs/promises` |
| `Number(process.env.PORT)` when the name is missing | You get `NaN` and may listen on a nonsense port | Test for `undefined` and exit with a message |

### Tip

`path.join` builds. `path.resolve` collapses `..` into an absolute path you can compare with the allowed root.

### Practice

Join a `notes.txt` path next to the current ES module and read it as UTF-8.

### Solution

```javascript
import { readFile } from "node:fs/promises";
import path from "node:path";
import { fileURLToPath } from "node:url";

const here = path.dirname(fileURLToPath(import.meta.url));
const file = path.join(here, "notes.txt");
const text = await readFile(file, "utf8");
console.log(text);
```

Result: prints the contents of `notes.txt` beside the module. The path does not depend on `process.cwd()`.

## 5. Events and errors

### Concept

Node's `EventEmitter` is a list of listeners. `on` registers one. `emit` calls every listener for that name, in registration order, with the arguments you pass. An error thrown in a promise, with nobody awaiting it, can crash the process. `await` it or attach `.catch`. An `error` event on a stream, with no listener, can crash the process too. An expected failure, such as a missing file, belongs in `try/catch` with a useful message. A programmer mistake should throw during development. A request handler catches its failure and sends a 500. Do not leak stack traces to the client in production.

### Terms

| Term | Meaning |
| --- | --- |
| Listener | A function registered for one event name |
| `emit` | Calls the listeners now, on the current stack |
| `"error"` | The special event. With no listener, Node throws it |
| Unhandled rejection | A promise that failed with nobody attached. Current Node exits |
| `error.code` | A string such as `ENOENT` on a filesystem error |

### Methods

| Failure | What you do |
| --- | --- |
| Expected, such as a missing file | `try/catch` and a useful message |
| Programmer mistake | Let it throw during development |
| Rejected promise | `await` it or attach `.catch`. Otherwise the process can crash |
| `"error"` on a stream or emitter | Register a listener. Otherwise the process can crash |
| Request handler | Catch it and send a 500. Do not leak stack traces to the client in production |

### Examples

**Case: `on` then `emit`**

```javascript
import { EventEmitter } from "node:events";

const bus = new EventEmitter();
bus.on("ready", (name) => {
  console.log(name);
});
bus.emit("ready", "api");
```

Result: prints `api`.

Why: `on` appends a listener. `emit` calls it synchronously with the extra arguments. Nothing is queued on the event loop. If you `emit` before `on`, that event is gone. `once` removes the listener after the first call.

**Case: `"error"` with no listener crashes**

```javascript
import { EventEmitter } from "node:events";

const bus = new EventEmitter();
bus.emit("error", new Error("disk"));
```

Result: the process throws `Error: disk` and can exit.

Why: `"error"` is not an ordinary name. An emitter or a stream that emits it with no listener throws. The same is true of a file stream. Add a listener before the work starts.

**Case: a listener keeps the process up**

```javascript
import { EventEmitter } from "node:events";

const bus = new EventEmitter();
bus.on("error", (error) => {
  console.error(error.message);
});
bus.emit("error", new Error("disk"));
```

Result: prints `disk`. The process does not throw.

Why: one `"error"` listener marks the event as handled. Log what you need and decide whether to continue. An unhandled `"error"` from a stream is a process crash, not a rejected promise.

**Case: a missing file, and a promise nobody awaited**

```javascript
import { readFile } from "node:fs/promises";

try {
  await readFile("missing.txt", "utf8");
} catch (error) {
  console.error(error.code);
}
```

```javascript
Promise.reject(new Error("boom"));
```

Result: the `readFile` call prints `ENOENT`. The rejected promise is an unhandled rejection. On current Node the process reports it and exits with a non-zero code.

Why: a missing file is expected, so `try/catch` can print a useful code. A programmer mistake, such as calling a function that does not exist, should throw during development so you see the stack. A request handler catches the failure and responds with status 500 and a short message. The stack trace stays in the server log. `await` the promise, or attach `.catch`, so it does not become an unhandled rejection.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `emit("error")` or a stream error with no listener | The process throws and can exit | `bus.on("error", handler)` before you start the work |
| `Promise.reject` or an async call with no `await` and no `.catch` | Unhandled rejection. Current Node exits non-zero | `await` it, or attach `.catch` |
| Sending `error.stack` in a 500 response | The client sees your file paths | Log the stack on the server. Send a short message |
| Catching a missing file and ignoring `error.code` | You hide permission errors and real bugs in the same branch | Branch on `ENOENT`. Rethrow the rest |

### Tip

`emit` runs listeners now. The event loop is involved only when the emitter is wrapping I/O, such as a stream's `"data"` event.

### Practice

Register a `"ready"` listener that prints the name, then emit `"ready"` with `"api"`. Also catch a missing file and print its code.

### Solution

```javascript
import { EventEmitter } from "node:events";
import { readFile } from "node:fs/promises";

const bus = new EventEmitter();
bus.on("ready", (name) => {
  console.log(name);
});
bus.emit("ready", "api");

try {
  await readFile("missing.txt", "utf8");
} catch (error) {
  console.error(error.code);
}
```

Result: prints `api`, then `ENOENT`.

## 6. HTTP without a framework

### Concept

`http.createServer` gives you a request and a response. You read `req.method` and `req.url`, then you write the status and headers with `res.writeHead` and the body with `res.end`. Headers must be written before the body. `req.url` is the path plus the query string. Express exists so you do not parse URLs and bodies by hand. Know this layer so Express is not magic. `server.listen` keeps the process alive because it is waiting for connections.

### Terms

| Term | Meaning |
| --- | --- |
| `req.method` | `GET`, `POST`, and the rest |
| `req.url` | Path plus query string. `/health?x=1` is not equal to `/health` |
| `res.writeHead` | Status and headers. Must happen before the body |
| `res.end` | Finish the response. You can pass the body here |
| Status | 200 ok, 201 created, 400 bad input, 401 unauthenticated, 403 forbidden, 404 missing, 500 server fault |

### Methods

| Piece | Meaning |
| --- | --- |
| `http.createServer(listener)` | One function for every request |
| `req.method` | The HTTP method |
| `req.url` | Path plus query string |
| `res.writeHead(status, headers)` | Status and headers, before any body byte |
| `res.end(body)` | Finish. The optional body is the payload |
| `server.listen(3000)` | Bind the port and keep the process running |
| `JSON.stringify` | How you build a JSON body yourself |

### Examples

**Case: `GET /health`**

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

Result: `GET /health` responds with status 200 and `{"status":"ok"}`. Any other path, such as `GET /missing`, responds with status 404 and `{"error":"not found"}`. The process stays up because it is waiting for connections.

Why: one listener sees every request. You branch on the method and the URL yourself. `return` after `end` so you do not write a second response. `writeHead` sets the status and the JSON content type before the body.

**Case: the query string is part of `req.url`**

```javascript
import http from "node:http";

const server = http.createServer((req, res) => {
  const path = req.url.split("?")[0];
  if (req.method === "GET" && path === "/health") {
    res.writeHead(200, { "content-type": "application/json" });
    res.end(JSON.stringify({ status: "ok" }));
    return;
  }
  res.writeHead(404, { "content-type": "application/json" });
  res.end(JSON.stringify({ error: "not found" }));
});

server.listen(3000);
```

Result: `GET /health?x=1` responds with 200. The same request compared with `req.url === "/health"` would have been a 404, because `req.url` is `/health?x=1`.

Why: Node does not split the query for you. The path is the text before `?`. Express later exposes `req.path` and `req.query` so this split is not your job.

**Case: headers after `end` throw**

```javascript
import http from "node:http";

const server = http.createServer((_req, res) => {
  res.writeHead(200, { "content-type": "text/plain" });
  res.end("ok");
  res.writeHead(201, { "content-type": "text/plain" });
});

server.listen(3000);
```

Result: the client can receive `ok` with status 200. The second `writeHead` throws `ERR_HTTP_HEADERS_SENT` on the server.

Why: `end` finishes the message. Status and headers must happen before the body. Build the status first, then call `end` once.

**Case: 201 and 400 are choices you write**

```javascript
import http from "node:http";

const server = http.createServer((req, res) => {
  if (req.method === "POST" && req.url === "/notes") {
    res.writeHead(400, { "content-type": "application/json" });
    res.end(JSON.stringify({ error: "text is required" }));
    return;
  }
  res.writeHead(201, { "content-type": "application/json" });
  res.end(JSON.stringify({ id: "1" }));
});

server.listen(3000);
```

Result: `POST /notes` responds with status 400 and `{"error":"text is required"}`. Any other request responds with status 201 and `{"id":"1"}`.

Why: the status codes are yours to choose. 200 is ok, 201 is created, 400 is bad input, 401 is unauthenticated, 403 is forbidden, 404 is missing, and 500 is a server fault. This handler does not read the body yet. You would collect `req` `"data"` chunks and `JSON.parse` them. Express does that in `express.json()`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Comparing `req.url` to a path when a query is present | `/health?x=1` misses the `/health` branch and returns 404 | Split on `?`, or parse with `URL` |
| `writeHead` after `end` | `ERR_HTTP_HEADERS_SENT` | Set status and headers, then `end` once |
| Forgetting `return` after a response | The function falls through and writes a second response, which throws | `return` after every `end` |
| Leaving the JSON content type unset | The client may treat the body as plain text | Pass `content-type: application/json` in `writeHead` |

### Tip

If you can explain `method`, `url`, `writeHead`, and `end`, the Express helpers in the next section are shortcuts for this same response.

### Practice

Respond to `GET /health` with status 200 and `{"status":"ok"}`, and send 404 for every other URL.

### Solution

```javascript
import http from "node:http";

const server = http.createServer((req, res) => {
  if (req.method === "GET" && req.url.split("?")[0] === "/health") {
    res.writeHead(200, { "content-type": "application/json" });
    res.end(JSON.stringify({ status: "ok" }));
    return;
  }
  res.writeHead(404, { "content-type": "application/json" });
  res.end(JSON.stringify({ error: "not found" }));
});

server.listen(3000);
```

Result: `GET /health` returns 200 and `{"status":"ok"}`. `GET /other` returns 404 and `{"error":"not found"}`.

## 7. Express routes

### Concept

Express maps a method and a path pattern to a function. `app.use(express.json())` parses a JSON body before the routes that read `req.body`. `res.json` sets the JSON content type and ends the response. `res.status(404).json(...)` sets the status first. Calling `res.json` twice throws. `return` after you send an error so the function does not fall through. A route param such as `:id` is a string on `req.params`. `express.urlencoded` parses HTML form bodies. `express.static("public")` serves files.

### Terms

| Term | Meaning |
| --- | --- |
| `app.get` | Register a reader for a path |
| Route param | `/homes/:id` puts the text in `req.params.id` |
| Query string | `/homes?city=Irvine` puts `Irvine` in `req.query.city` |
| `req.body` | The parsed body, only after `express.json()` or `express.urlencoded` |
| `res.json` | Set the JSON content type, send the body, and end the response |

### Methods

| Method helper | Typical use |
| --- | --- |
| `app.get` | Read |
| `app.post` | Create |
| `app.put` | Replace |
| `app.patch` | Partial update |
| `app.delete` | Remove |

| Source | Example | Lives in |
| --- | --- | --- |
| Route param | `/homes/:id` | `req.params.id` |
| Query string | `/homes?city=Irvine` | `req.query.city` |
| JSON body | `{ "city": "Irvine" }` | `req.body`, after `express.json()` |
| Header | `Authorization` | `req.get("authorization")` |

| Job | `node:http` | Express |
| --- | --- | --- |
| Method | `req.method` | `req.method`, and you register `app.get` or `app.post` |
| Path | `req.url` includes the query string. You split it | `req.path` is the path. `req.query` is parsed |
| Route param | You match the path yourself | `req.params.id` from `/homes/:id` |
| Body | Read the stream and `JSON.parse` it | `req.body`, after `express.json()` |
| Header | `req.headers["content-type"]` | `req.get("content-type")` |
| Status and body | `res.writeHead(status, headers)` then `res.end(body)` | `res.status(status).json(body)` |
| Send a file | Set headers and stream the file | `res.sendFile(path)` |

### Examples

**Case: health, a param, and a JSON body**

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

Result: `GET /health` returns 200 and `{"status":"ok"}`. `GET /homes/42` returns `{"id":"42"}`. `POST /homes` with `{ "city": "Irvine" }` returns 201 and `{"city":"Irvine"}`. `POST /homes` with no city returns 400 and `{"error":"city is required"}`.

Why: `express.json()` runs first and fills `req.body`. `req.params.id` is the string from the path. `res.json` sets the content type and ends the response. `res.status(400)` sets the status before that. The `return` stops the 400 branch from also sending 201.

**Case: query and header**

```javascript
import express from "express";

const app = express();

app.get("/homes", (req, res) => {
  res.json({
    city: req.query.city ?? null,
    auth: req.get("authorization") ?? null,
  });
});

app.listen(3000);
```

Result: `GET /homes?city=Irvine` with header `Authorization: Bearer abc` returns `{"city":"Irvine","auth":"Bearer abc"}`. `req.path` is `/homes`. The query is not part of `req.path`.

Why: Express splits what `req.url` jammed together. `req.query.city` is the query. `req.get` reads a header by name, without you indexing `req.headers` yourself. On the raw server, `req.headers["content-type"]` is the matching lookup, and the name is lowercase.

**Case: `res.json` twice throws**

```javascript
import express from "express";

const app = express();

app.get("/homes/:id", (req, res) => {
  if (req.params.id === "0") {
    res.status(400).json({ error: "bad id" });
  }
  res.json({ id: req.params.id });
});

app.listen(3000);
```

Result: `GET /homes/0` sends the 400 body, then throws because `res.json` runs again and the headers were already sent. `GET /homes/5` returns `{"id":"5"}` once.

Why: `res.json` ends the response the way `res.end` does. Calling it twice throws. `return` after you send an error. The param is still a string, so check `=== "0"`, not `=== 0`.

**Case: JSON middleware order, forms, and static files**

```javascript
import express from "express";

const app = express();

app.post("/early", (req, res) => {
  res.json({ city: req.body?.city ?? null });
});

app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(express.static("public"));

app.post("/late", (req, res) => {
  res.json({ city: req.body?.city ?? null });
});

app.listen(3000);
```

Result: `POST /early` with a JSON body responds with `{"city":null}`, because `express.json()` has not run yet. `POST /late` with `{ "city": "Irvine" }` responds with `{"city":"Irvine"}`. A `public/house.jpg` file is served at `/house.jpg`. An HTML form body is parsed only where `express.urlencoded` has already run.

Why: middleware runs in the order you `use` it, and only for routes registered after that `use`. `req.body` stays empty until a parser runs. `express.static("public")` serves files and ends the response when it finds one. `res.sendFile(path)` is the explicit version when you want one file from a route.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Reading `req.body` before `express.json()` | `req.body` is `undefined` and the handler misses fields | `app.use(express.json())` before the routes that parse JSON |
| No `return` after `res.status(400).json(...)` | A second `res.json` throws | `return` immediately after the error response |
| Comparing `req.params.id` to a number | `"42" === 42` is false, so the branch never matches | Compare strings, or convert with `Number` after a format check |
| `express.static` mounted after a catch-all that sends a response | The file is never reached | Mount static files before the fallback, or call them in an order that can still run |

### Tip

`app.get` reads, `app.post` creates, `app.put` replaces, `app.patch` updates part of a record, and `app.delete` removes it. The path pattern is the same idea for each.

### Practice

Write `GET /health` and `POST /notes`. Return 400 when `text` is missing.

### Solution

```javascript
import express from "express";

const app = express();
app.use(express.json());

app.get("/health", (_req, res) => {
  res.json({ status: "ok" });
});

app.post("/notes", (req, res) => {
  const text = req.body?.text;
  if (typeof text !== "string" || text.trim() === "") {
    res.status(400).json({ error: "text is required" });
    return;
  }
  res.status(201).json({ text: text.trim() });
});

app.listen(3000);
```

Result: `GET /health` returns 200 and `{"status":"ok"}`. `POST /notes` without `text` returns 400 and `{"error":"text is required"}`. `POST /notes` with `{ "text": "hello" }` returns 201 and `{"text":"hello"}`.

## 8. Middleware, routers, and errors

### Concept

Middleware is a function of `(req, res, next)`. It runs in the order you `use` it. It either sends a response or calls `next()`. If it does neither, the request hangs. Error middleware has four arguments, `(error, req, res, next)`, and it runs only when something calls `next(error)` or when a synchronous handler throws. Register it after the routes. Express 4 does not forward a rejected promise from an `async` handler. Express 5 does. A small wrapper that calls `next(error)` works on both. A router is a mini app you mount on a path. Validate input on the server. Do not reflect stack traces. Keep tokens in the environment. Limit body size. Browsers block other origins unless you send CORS headers. A server-to-server call is not subject to CORS.

### Terms

| Term | Meaning |
| --- | --- |
| Middleware | `(req, res, next)`. Runs in `use` order |
| `next()` | Hand the request to the next matching function |
| `next(error)` | Skip to error middleware |
| Error middleware | Four arguments: `(error, req, res, next)`. Express tells it apart by the count |
| Router | A mini app. `app.use("/homes", homes)` prefixes its paths |
| CORS | Browser rule. The server sends headers that allow another origin. Server-to-server calls ignore it |

### Methods

| Middleware job | Example |
| --- | --- |
| Parse the body | `express.json()` |
| Log | Print method and URL, then `next()` |
| Authenticate | Read a session. `401` if there is none. `next()` if there is |
| Authorize | `403` when the role is wrong |
| Serve files | `express.static` |
| Handle errors | Four arguments: `(error, req, res, next)` |
| Limit size | `express.json({ limit: "100kb" })` |
| Share a prefix | `express.Router()`, then `app.use("/homes", homes)` |

| Practice | Why |
| --- | --- |
| Validate input on the server | The client can send anything |
| Do not reflect stack traces | They describe your files |
| Keep tokens in the environment | A committed key is public |
| Limit body size | `express.json({ limit: "100kb" })` |
| CORS | Browsers block other origins unless you send the headers. A server-to-server call is not subject to CORS |

### Examples

**Case: order is the order of `use`**

```javascript
import express from "express";

const app = express();

function requireJson(req, res, next) {
  if (req.method === "POST" && !req.is("application/json")) {
    res.status(415).json({ error: "send JSON" });
    return;
  }
  next();
}

app.use(requireJson);
app.use(express.json({ limit: "100kb" }));

app.post("/notes", (req, res) => {
  res.status(201).json({ text: req.body?.text ?? "" });
});

app.listen(3000);
```

Result: a `POST /notes` whose content type is not JSON gets 415 and `{"error":"send JSON"}`, and `express.json()` does not run for that request. A JSON post within 100kb reaches the route. A larger JSON body is rejected with status 413.

Why: middleware runs in the order you `use` it. `requireJson` either sends 415 or calls `next()`. `express.json({ limit: "100kb" })` then parses, and it refuses a body over the limit. Skipping `next()` without a response leaves the client waiting.

**Case: forgetting `next` hangs**

```javascript
import express from "express";

const app = express();

app.use((_req, _res, _next) => {
  console.log("seen");
});

app.get("/health", (_req, res) => {
  res.json({ status: "ok" });
});

app.listen(3000);
```

Result: every request prints `seen` and then waits. `/health` never sends `{"status":"ok"}`.

Why: the first function did not send a response and did not call `next()`. Express has no later function to run. Log middleware should print the method and URL and then call `next()`. Auth middleware sends 401 when the session is missing, sends 403 when the role is wrong, and calls `next()` when the caller is allowed.

**Case: four arguments catch `next(error)`**

```javascript
import express from "express";

const app = express();

app.get("/boom", (_req, _res, next) => {
  next(new Error("disk"));
});

app.use((error, _req, res, _next) => {
  console.error(error);
  res.status(500).json({ error: "failed" });
});

app.listen(3000);
```

Result: `GET /boom` logs the `Error` on the server and responds with status 500 and `{"error":"failed"}`. The client does not receive the stack.

Why: Express treats a four-argument function as error middleware. A three-argument function is ordinary middleware and will not catch `next(error)`. The error function has to be registered after the routes. `next(error)` skips ordinary middleware and looks forward for the four-argument one. Log `error` on the server. Send a short body.

**Case: async failures and a router**

```javascript
import express from "express";

const app = express();

function asyncHandler(fn) {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

const homes = express.Router();

homes.get("/", asyncHandler(async (_req, res) => {
  const rows = await loadHomes();
  res.json(rows);
}));

app.use("/homes", homes);

app.use((error, _req, res, _next) => {
  console.error(error);
  res.status(500).json({ error: "failed" });
});

app.listen(3000);
```

Result: `GET /homes` runs the router. If `loadHomes()` rejects, `asyncHandler` calls `next(error)` and the client receives 500 and `{"error":"failed"}`. The stack stays in the server log.

Why: an async handler's rejection does not reach the error middleware on Express 4 unless you pass it on. Express 5 forwards that rejection for you. The wrapper works on both. A router is a mini app. `homes.get("/")` plus `app.use("/homes", homes)` is `GET /homes`. Validate the input on that route. The client can send anything. CORS headers matter when a browser on another origin calls you. A server-to-server call is not subject to CORS.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Middleware that neither responds nor calls `next()` | The request hangs | `next()`, or send a status and `return` |
| Error handler with three parameters | Express never treats it as error middleware | `(error, req, res, next)`, registered last |
| `async` route that throws, on Express 4, with no wrapper | The rejection never reaches the four-argument handler. The request can hang | `asyncHandler`, or upgrade to Express 5 and still send a safe 500 |
| Mounting the router at `/homes` and also writing `/homes` inside the router | The public path becomes `/homes/homes` | The router path is the suffix. Use `homes.get("/")` |

### Tip

Read the middleware list from top to bottom. The first function that sends a response ends the chain. `next(error)` jumps to the four-argument function at the bottom.

### Practice

In one sentence, say what `next` is for. Then mount a router so `GET /homes` returns an empty JSON array.

### Solution

`next` hands the request to the next middleware or route. `next(error)` hands a failure to the four-argument error middleware. If you already sent the response, do not call `next`.

```javascript
import express from "express";

const app = express();
const homes = express.Router();

homes.get("/", (_req, res) => {
  res.json([]);
});

app.use("/homes", homes);

app.use((error, _req, res, _next) => {
  console.error(error);
  res.status(500).json({ error: "failed" });
});

app.listen(3000);
```

Result: `GET /homes` returns 200 and `[]`. A later `next(error)` returns 500 and `{"error":"failed"}` without the stack in the body.

## Drill

1. Read `process.argv` and print every word after the script name.
2. Write `GET /health` and `POST /notes` that returns 400 when `text` is missing.
3. Say what `next` is for, in one sentence.
4. Say why a long `for` loop in a request handler blocks other requests, and `await readFile` does not.

### Solution 1

The practice solution in section 1.

```javascript
const args = process.argv.slice(2);
console.log(args.join(" "));
```

Result: `node app.js one two` prints `one two`. `process.argv[0]` is `node` and `process.argv[1]` is the script, so `slice(2)` keeps only the words after the script name.

### Solution 2

The practice solution in section 7.

```javascript
import express from "express";

const app = express();
app.use(express.json());

app.get("/health", (_req, res) => {
  res.json({ status: "ok" });
});

app.post("/notes", (req, res) => {
  const text = req.body?.text;
  if (typeof text !== "string" || text.trim() === "") {
    res.status(400).json({ error: "text is required" });
    return;
  }
  res.status(201).json({ text: text.trim() });
});

app.listen(3000);
```

Result: `GET /health` returns 200 and `{"status":"ok"}`. `POST /notes` with no `text` returns 400 and `{"error":"text is required"}`.

### Solution 3

The practice solution in section 8. `next` passes the request to the next middleware or route. Call `next(error)` to reach error middleware, which is the function with four arguments: `(error, req, res, next)`.

Result: a middleware that sends a response does not call `next()`. A middleware that does neither hangs the request.

### Solution 4

The practice solution in section 2. A long `for` loop stays on the call stack, so the event loop cannot run another request, a timer macrotask, or a file callback. `await readFile` pauses that function and lets other requests run while the disk works. `await` does not put the thread to sleep.

Result: during the `await`, other requests can run, and the file arrives as one string. During the loop, they cannot.
