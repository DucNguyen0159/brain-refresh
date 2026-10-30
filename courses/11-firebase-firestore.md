# Firebase and Firestore

Course 11 of 15. Previous: [10-sql.md](10-sql.md). Next: [12-tailwind.md](12-tailwind.md).

Firebase is a set of hosted services: login, a document database, file storage, messaging, and hosting. Firestore is the document database. It is not SQL, and it is not file storage. Cloud Storage is the file service.

Every numbered section has the same parts: the concept, the terms, the methods, several cases with the result, common mistakes, a tip, and a practice with a solution.

## Contents

1. [The project and the client](#1-the-project-and-the-client)
2. [Authentication](#2-authentication)
3. [Documents and collections](#3-documents-and-collections)
4. [Reads and queries](#4-reads-and-queries)
5. [Writes](#5-writes)
6. [Security rules](#6-security-rules)
7. [Messaging, storage, and hosting](#7-messaging-storage-and-hosting)

## 1. The project and the client

### Concept

A Firebase project has a config object. The values in that object identify the project. They are not a secret in the way a service-account private key is. Security rules are the lock. The web app calls `initializeApp` with that config, then asks the same app object for Auth and Firestore.

### Terms

| Name | Role |
| --- | --- |
| Web config | Safe to ship in a web app, because rules restrict it |
| `apiKey` | Identifies the web client. Rules still decide what that client may read or write |
| Service account | A private key for servers and admin scripts. Never in the browser |
| Emulator | A local Auth and Firestore. Use it for tests |
| `projectId` | The project the client talks to |

### Methods

| Method | Effect |
| --- | --- |
| `initializeApp(config)` | Builds the app handle from the web config |
| `getAuth(app)` | The Auth handle for that app |
| `getFirestore(app)` | The Firestore handle for that app |
| `connectAuthEmulator` / `connectFirestoreEmulator` | Point those handles at the local emulator during tests |

### Examples

**Case: one app, then Auth and Firestore**

```javascript
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getFirestore } from "firebase/firestore";

const app = initializeApp({
  apiKey: "your-web-api-key",
  authDomain: "your-app.firebaseapp.com",
  projectId: "your-app",
  appId: "your-app-id",
});

const auth = getAuth(app);
const db = getFirestore(app);
```

Result: the SDK returns an Auth handle and a Firestore handle bound to `your-app`.
Why: `initializeApp` only identifies the project. `getAuth` and `getFirestore` are the services you call later.

**Case: the web config is in the shipped bundle**

```javascript
const app = initializeApp({
  apiKey: "your-web-api-key",
  authDomain: "your-app.firebaseapp.com",
  projectId: "your-app",
  appId: "your-app-id",
});
```

Result: the browser can open the project. It still cannot read or write a document the rules forbid.
Why: the config names the project. Security rules are the lock, so shipping the web config is expected.

**Case: a service account stays off the page**

```javascript
// server only
import { initializeApp, cert } from "firebase-admin/app";

initializeApp({
  credential: cert(serviceAccount),
  projectId: "your-app",
});
```

Result: the Admin SDK signs in as the project, which bypasses client security rules.
Why: a service-account private key is a server credential. Putting that JSON in the browser gives away the project.

**Case: tests talk to the emulator**

```javascript
import { connectAuthEmulator } from "firebase/auth";
import { connectFirestoreEmulator } from "firebase/firestore";

connectAuthEmulator(auth, "http://127.0.0.1:9099");
connectFirestoreEmulator(db, "127.0.0.1", 8080);
```

Result: later Auth and Firestore calls hit the local emulator instead of the hosted project.
Why: the emulator is a local Auth and Firestore for tests. Connect it before those calls.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Treating the web `apiKey` as a private password | You hide a value the browser is supposed to have, and you still have no lock | Ship the web config and write security rules |
| Importing a service-account JSON into the web bundle | Anyone can take the private key and use the Admin SDK | Keep that file on the server |
| Calling `getFirestore()` on a different `initializeApp` than `getAuth()` | Auth and documents point at two projects | Pass the same `app` into both |
| Running tests against the hosted project | Tests write real documents | Connect the Auth and Firestore emulators |

### Tip

If a read fails after a successful `initializeApp`, inspect the rules before you rotate the web config.

### Practice

Initialize an app from a web config and create both handles. Say which of the two credentials may live in that file.

### Solution

```javascript
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getFirestore } from "firebase/firestore";

const app = initializeApp({
  apiKey: "your-web-api-key",
  authDomain: "your-app.firebaseapp.com",
  projectId: "your-app",
  appId: "your-app-id",
});

const auth = getAuth(app);
const db = getFirestore(app);
```

The web config may live in that file. The service-account private key may not.

## 2. Authentication

### Concept

Auth tracks a session on this device. A signed-in session is a user object. A signed-out session is null. Email and password calls return a promise. Failed calls reject. Wrong password and unknown email are errors you show in the form. `user.uid` is the stable id. Use it as the document id for that person's profile.

### Terms

| Term | Meaning |
| --- | --- |
| User | The signed-in account. `null` means signed out |
| `uid` | The stable id. Use it as the document id for that person's profile |
| Session | The signed-in state on this device |
| Provider | Email and password, or another provider such as Google |
| Rejection | A failed call. Show it on the form |

### Methods

| Call | Result |
| --- | --- |
| `createUserWithEmailAndPassword(auth, email, password)` | A new user, and a session |
| `signInWithEmailAndPassword(auth, email, password)` | A session for an existing user |
| `signOut(auth)` | Clears the session on this device |
| `onAuthStateChanged(auth, callback)` | Runs now and whenever the session changes |
| `sendPasswordResetEmail(auth, email)` | A reset email |
| `signInWithPopup(auth, provider)` | A session from another provider. The session shape is still a user or null |

### Examples

**Case: create an account**

```javascript
import { createUserWithEmailAndPassword } from "firebase/auth";

const cred = await createUserWithEmailAndPassword(auth, email, password);
console.log(cred.user.uid);
```

Result: the SDK creates a user and a session, and `uid` is that user's stable id.
Why: create both registers the account and signs this device in. Store the profile at `users/{uid}`.

**Case: sign in, then sign out on this device**

```javascript
import { signInWithEmailAndPassword, signOut } from "firebase/auth";

await signInWithEmailAndPassword(auth, email, password);
await signOut(auth);
```

Result: the first call starts a session for an existing user. The second clears the session on this device.
Why: `signOut` affects this device. It does not delete the account.

**Case: the listener runs immediately**

```javascript
import { onAuthStateChanged } from "firebase/auth";

onAuthStateChanged(auth, (user) => {
  if (user) console.log(user.uid);
  else console.log("signed out");
});
```

Result: the callback runs once with the current user or null, then again whenever the session changes.
Why: the UI can paint the signed-in and signed-out states from this one listener.

**Case: a wrong password rejects**

```javascript
import { signInWithEmailAndPassword } from "firebase/auth";

try {
  await signInWithEmailAndPassword(auth, email, password);
} catch (error) {
  formError.textContent = "Email or password is wrong.";
}
```

Result: the SDK rejects the promise. The form shows the message. The page does not crash.
Why: unknown email and wrong password are form errors. Catch them beside the fields.

**Case: Google uses the same session shape**

```javascript
import { GoogleAuthProvider, signInWithPopup } from "firebase/auth";

const cred = await signInWithPopup(auth, new GoogleAuthProvider());
console.log(cred.user.uid);
```

Result: after the popup, `cred.user` is a user object with a `uid`, or the call rejects if the person dismisses it.
Why: other providers use `signInWithPopup` and a provider object. The session is still a user or null.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Letting a rejected sign-in become an uncaught error | The UI looks like a crash | Catch the promise and show the message on the form |
| Reading `user.uid` before the listener runs | `user` is still unknown | Wait for `onAuthStateChanged` |
| Using the email as the profile document id | The id changes if the email changes | Use `uid` |
| Calling `signOut` and expecting the account to vanish | The account remains. Only this device's session clears | Delete the user only from a flow that means deletion |

### Tip

Subscribe to `onAuthStateChanged` once near the root. The callback is how you learn the session, including the session that already existed on load.

### Practice

Sign an existing user in, print `uid`, and send a password reset email for that same address.

### Solution

```javascript
import {
  sendPasswordResetEmail,
  signInWithEmailAndPassword,
} from "firebase/auth";

const cred = await signInWithEmailAndPassword(auth, email, password);
console.log(cred.user.uid);
await sendPasswordResetEmail(auth, email);
```

Result: the SDK starts a session, prints the stable id, and sends the reset email.

## 3. Documents and collections

### Concept

Firestore stores documents inside collections. A document is a set of fields with an id. A collection is a list of documents. A document may contain a subcollection. The database will not force a schema. A field exists because your code writes it. Document ids you choose, such as the user id, make `users/{uid}` a direct read. Auto ids, from `addDoc`, are for records the user will have many of, such as orders. Keep documents small. A growing list that can become huge belongs in a subcollection, not in one unbounded array.

### Terms

| SQL idea | Firestore idea |
| --- | --- |
| Table | Collection |
| Row | Document |
| Primary key | Document id |
| Column that every row must have | A field you agree to write. The database will not force a schema |
| Join | Store the other id, or copy the few fields you must show |
| Nested rows | A subcollection, or an array inside the document |

| Field type | Example |
| --- | --- |
| string, number, boolean | `"Irvine"`, `999000`, `true` |
| array | `["a", "b"]` |
| map | `{ city: "Irvine", zip: "92602" }` |
| timestamp | `serverTimestamp()` |
| reference | A pointer at another document |
| null | A field that is present and empty |

### Methods

| Method | Effect |
| --- | --- |
| `doc(db, "users", uid)` | A reference to one document. You chose the id |
| `collection(db, "orders")` | A reference to a collection |
| `doc(db, "users", uid, "carts", "active")` | A document inside a subcollection |
| `addDoc(collection(db, "orders"), data)` | Creates a document and lets Firestore choose the id |
| `serverTimestamp()` | A timestamp field filled with the server clock |

### Examples

**Case: a profile id you choose**

```text
users/{uid}
  displayName
  role
```

```javascript
const ref = doc(db, "users", uid);
```

Result: the SDK points at one document path, `users/{uid}`. A later `getDoc` is a direct read.
Why: the user id is the document id, so you do not query to find that person's profile.

**Case: many orders, ids chosen by Firestore**

```javascript
import { addDoc, collection } from "firebase/firestore";

const ref = await addDoc(collection(db, "orders"), {
  userId: uid,
  status: "pending",
  total: 40,
});
console.log(ref.id);
```

Result: the SDK creates a new document under `orders` and returns the auto id.
Why: one person has many orders. `addDoc` is the call for that. The profile still uses the uid you chose.

**Case: a cart that would grow forever**

```text
users/{uid}
  displayName
  role

users/{uid}/carts/active
  items: [{ artworkId, qty }]
```

Result: the cart is its own document in a subcollection. The user document stays the two profile fields.
Why: a list that can become huge belongs in a subcollection. An unbounded array on `users/{uid}` grows the profile without a limit.

**Case: field types you can store**

```javascript
import { doc, serverTimestamp, setDoc } from "firebase/firestore";

await setDoc(doc(db, "orders", orderId), {
  userId: uid,
  status: "pending",
  total: 999000,
  active: true,
  tags: ["a", "b"],
  address: { city: "Irvine", zip: "92602" },
  createdAt: serverTimestamp(),
  note: null,
});
```

Result: the SDK writes a string, a number, a boolean, an array, a map, a server timestamp, and a present-but-empty field.
Why: those are document fields. A reference field would point at another document. Firestore does not require every order to have the same fields.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Expecting Firestore to reject a missing field | The write succeeds. The field is simply absent | Check the shape in your code, or in rules |
| Putting every order inside the user document as one array | The document grows without a bound and rewrites get large | A subcollection, or an `orders` collection that stores `userId` |
| Querying to find `users/{uid}` | An extra query for a document you can name | `doc(db, "users", uid)` |
| Using `addDoc` for the profile | Each call creates another profile with a random id | `setDoc` on `users/{uid}` |

### Tip

Copy a field only when the screen must show it without a second read. Store the other id when the screen can look the rest up.

### Practice

Sketch `users/{uid}` and one order document. Give the order an auto id and a `userId` field.

### Solution

```text
users/{uid}
  displayName
  role

orders/{orderId}
  userId
  status
  total
```

`orderId` comes from `addDoc`. `uid` is the id you chose for the profile.

## 4. Reads and queries

### Concept

`getDoc` reads one document. `getDocs` reads every document in a query. `exists()` is false when the id is missing, and the promise still fulfills. A permission error rejects the promise. A missing index is a third outcome: the error message includes a link that creates it. Equality filters on several fields are fine. Range filters on two different fields are the case that needs planning. `onSnapshot` listens. The callback runs with the current result and again when it changes. Unsubscribe when the component goes away.

### Terms

| | `getDoc` / `getDocs` | `onSnapshot` |
| --- | --- | --- |
| What you get | One promise of the current result | A callback now, and again on each change |
| Missing document | `exists()` is false. The promise still fulfills | The snapshot's `exists()` is false |
| Permission denied | The promise rejects | The error callback runs |
| How long it lasts | One read | Until you call the function it returned |
| When to use it | A page that can load once | A screen that must follow edits |

### Methods

| Call | Result |
| --- | --- |
| `getDoc(ref)` | One document. `exists()` is false when the id is missing |
| `getDocs(query)` | Every document in a query |
| `where(field, op, value)` | A filter. `==`, `<`, `<=`, `>`, `>=`, `!=`, `in`, `array-contains` |
| `orderBy(field)` | Sort. Inequality filters usually need the same field ordered first |
| `limit(n)` | A maximum count |
| `startAfter(lastSnap)` | The next page, using the last document of the previous page |
| `onSnapshot(ref, onNext, onError)` | The current snapshot, then each change. Returns the unsubscribe function |

### Examples

**Case: one profile, present or missing**

```javascript
import { doc, getDoc } from "firebase/firestore";

const snap = await getDoc(doc(db, "users", uid));
if (snap.exists()) {
  const profile = snap.data();
  console.log(profile.displayName);
} else {
  console.log("no profile");
}
```

Result: the SDK fulfills with a snapshot. `exists()` is false when that id is missing, and the promise still fulfills.
Why: a missing document is an empty snapshot. It is a different outcome from a rule denial.

**Case: ten homes in one city, cheapest first**

```javascript
import { collection, getDocs, limit, orderBy, query, where } from "firebase/firestore";

const homes = query(
  collection(db, "homes"),
  where("city", "==", "Irvine"),
  where("price", "<=", 1500000),
  orderBy("price"),
  limit(10),
);
const list = await getDocs(homes);
list.forEach((row) => {
  console.log(row.id, row.data());
});
```

Result: the SDK returns at most ten documents in that city at or under the price, sorted by price ascending.
Why: `where` filters, `orderBy("price")` sorts, and the inequality is on the same field the query orders by. `limit(10)` caps the count.

**Case: the next page**

```javascript
import { collection, getDocs, limit, orderBy, query, startAfter, where } from "firebase/firestore";

const homes = query(
  collection(db, "homes"),
  where("city", "==", "Irvine"),
  orderBy("price"),
);
const first = await getDocs(query(homes, limit(10)));
const last = first.docs[first.docs.length - 1];
const next = await getDocs(query(homes, startAfter(last), limit(10)));
```

Result: the second call returns the following page, starting after the last document of the first page.
Why: `startAfter` uses that snapshot as the cursor. It skips rows by position, not by guessing the next price.

**Case: a listener, then unsubscribe**

```javascript
import { doc, onSnapshot } from "firebase/firestore";

const stop = onSnapshot(
  doc(db, "orders", orderId),
  (snap) => {
    console.log(snap.exists() ? snap.data() : "missing");
  },
  (error) => {
    console.error(error.code);
  },
);
stop();
```

Result: the SDK runs the first callback with the current snapshot, runs it again on each change, and runs the error callback if the rules deny the read. `stop()` ends the listener.
Why: `onSnapshot` lasts until you call the function it returned. A permission error does not show up as `exists() === false`.

**Case: a missing index**

```javascript
const homes = query(
  collection(db, "homes"),
  where("city", "==", "Irvine"),
  where("beds", ">=", 3),
  where("price", "<=", 1500000),
  orderBy("price"),
);
await getDocs(homes);
```

Result: the SDK rejects the promise when this compound query has no index. The error message includes a link that creates it.
Why: equality filters on several fields are fine. Range filters on two different fields are the case that needs planning.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Treating `exists() === false` as permission denied | You show "missing" for a document the rules hid | A missing document fulfills. A denial rejects, or hits the error callback |
| Ordering by a field other than the inequality | The query fails until the order matches the range field | `orderBy` the inequality field first |
| Leaving `onSnapshot` running after the component is gone | The callback keeps firing | Call the unsubscribe function |
| Paging with a guessed price instead of `startAfter` | Rows are skipped or repeated when prices tie | Pass the last snapshot to `startAfter` |

### Tip

Read the error text. A missing index, a rule denial, and a missing document are three different outcomes.

### Practice

Write a query for homes in one city with a price at or under a cap, cheapest first, ten rows. Then say what `exists()` means on `getDoc`.

### Solution

```javascript
const homes = query(
  collection(db, "homes"),
  where("city", "==", "Irvine"),
  where("price", "<=", 1500000),
  orderBy("price"),
  limit(10),
);
const list = await getDocs(homes);
```

`getDoc` still fulfills when the id is missing. `exists()` is false in that case. A rule denial rejects instead.

## 5. Writes

### Concept

A write names a document and the fields to store. `setDoc` replaces the document. `setDoc` with `{ merge: true }` creates or patches. `updateDoc` patches a document that must already exist. `addDoc` creates a new document and picks the id. `deleteDoc` removes the document and does not remove subcollections. A batch cannot read. A transaction can. Use a transaction when the new value depends on the current value, such as "approve only if the status is still pending."

### Terms

| | `setDoc` | `setDoc` with `{ merge: true }` | `updateDoc` | `addDoc` |
| --- | --- | --- | --- | --- |
| Who picks the id | You | You | You. The document must already exist | Firestore |
| Document missing | Creates it | Creates it | Fails | Creates a new one |
| Fields you do not mention | Deleted | Kept | Kept | There are no older fields |
| A whole replacement | Yes | No | No | It is a new document |

### Methods

| Call | Effect |
| --- | --- |
| `setDoc(ref, data)` | Create or replace the whole document |
| `setDoc(ref, data, { merge: true })` | Create or patch. Unmentioned fields stay |
| `updateDoc(ref, data)` | Patch. Fails if the document is missing |
| `addDoc(collectionRef, data)` | Create with an auto id |
| `deleteDoc(ref)` | Remove the document. Does not remove subcollections |
| `serverTimestamp()` | The server's clock, so clients cannot lie about the time |
| `arrayUnion` / `arrayRemove` | Add or remove an array element without rewriting the array |
| `increment(1)` | Change a number without reading it first |
| Write batch | Several writes that succeed or fail together |
| Transaction | Read, then write, and retry if the documents changed |

### Examples

**Case: `setDoc` replaces the profile**

```javascript
import { doc, serverTimestamp, setDoc } from "firebase/firestore";

await setDoc(doc(db, "users", uid), {
  displayName: "Henry",
  role: "artist",
  createdAt: serverTimestamp(),
});
```

Result: the SDK writes exactly those fields. Any older field on that document is gone. `createdAt` becomes the server clock.
Why: plain `setDoc` is a whole replacement. `serverTimestamp()` keeps the client from choosing the time.

**Case: merge keeps fields you omit**

```javascript
await setDoc(
  doc(db, "users", uid),
  { displayName: "Henry" },
  { merge: true },
);
```

Result: the SDK updates `displayName` and leaves `role` and `createdAt` in place. If the document was missing, it creates it.
Why: `{ merge: true }` is a patch that can also create. Fields you do not mention are kept.

**Case: `updateDoc` requires the document**

```javascript
import { doc, updateDoc } from "firebase/firestore";

await updateDoc(doc(db, "orders", orderId), {
  status: "approved",
});
```

Result: the SDK changes `status` and keeps the other fields when the order exists. The promise rejects when the id is missing.
Why: `updateDoc` is a patch of a document that must already exist. It is the wrong call for a first write.

**Case: auto id, delete, and a counter**

```javascript
import { addDoc, arrayUnion, collection, deleteDoc, doc, increment, updateDoc } from "firebase/firestore";

const ref = await addDoc(collection(db, "orders"), {
  userId: uid,
  status: "pending",
  total: 40,
});
await updateDoc(ref, {
  tags: arrayUnion("rush"),
  edits: increment(1),
});
await deleteDoc(ref);
```

Result: the SDK creates an order with its own id, adds one array element, bumps a number, then deletes that document.
Why: `arrayUnion` and `increment` change those fields without a read. `deleteDoc` removes the order document and leaves any subcollection in place.

**Case: a transaction reads before it writes**

```javascript
import { doc, runTransaction } from "firebase/firestore";

await runTransaction(db, async (tx) => {
  const ref = doc(db, "orders", orderId);
  const snap = await tx.get(ref);
  if (snap.data().status !== "pending") return;
  tx.update(ref, { status: "approved" });
});
```

Result: the SDK approves the order only when the status it just read is still `pending`. If another write lands first, it retries.
Why: a batch cannot read. A transaction can. The new value here depends on the current value.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Using plain `setDoc` to change one field | Every omitted field is deleted | `{ merge: true }`, or `updateDoc` |
| Using `updateDoc` to create a profile | The promise rejects because the document is missing | `setDoc` for the first write |
| Expecting `deleteDoc` to clear subcollections | The child documents remain | Delete those documents too |
| Approving inside a batch after a separate read | The status can change between the read and the write | A transaction |

### Tip

Use `increment` and `arrayUnion` when you do not need the old value in your own code. Use a transaction when you do.

### Practice

Create an order with `addDoc`, then set `status` to `"approved"` only if it is still `"pending"`.

### Solution

```javascript
const ref = await addDoc(collection(db, "orders"), {
  userId: uid,
  status: "pending",
  total: 40,
});

await runTransaction(db, async (tx) => {
  const snap = await tx.get(ref);
  if (snap.data().status !== "pending") return;
  tx.update(ref, { status: "approved" });
});
```

The transaction reads the status it is about to change. A batch cannot do that read.

## 6. Security rules

### Concept

Rules run on the server for every client request. A check that lives only in the client can be skipped by a modified app. Rules cannot. They are not filters. A query must be allowed for every document it might return. Firestore does not silently drop the forbidden ones. Test rules with the emulator. A rule you have not tried will deny a legitimate write or allow a bad one.

### Terms

| Name | Meaning |
| --- | --- |
| `request.auth` | The signed-in user, or null |
| `request.auth.uid` | Their id |
| `request.resource.data` | The document as it will be after this write |
| `resource.data` | The document as it is now |
| `allow read` | Get and list |
| `allow write` | Create, update, and delete. Split them when the checks differ |
| Custom claim | An admin flag set from a server. The client cannot grant it |

### Methods

| Rule shape | Effect |
| --- | --- |
| `allow read, write: if request.auth != null && request.auth.uid == userId` | The signed-in user may read and write only their own `users/{userId}` document |
| `allow read: if request.auth.uid == resource.data.userId` | Read the order only when the stored `userId` is the caller |
| `allow create: if request.auth.uid == request.resource.data.userId` | Create only an order stamped with the caller's uid |
| `allow update: if request.resource.data.status == resource.data.status` | An update that must leave `status` unchanged |
| `match /orders/{orderId}` | One rule block for every document in `orders` |

### Examples

**Case: the owner rule**

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

Result: the server allows a signed-in client to read and write `users/{their uid}` and denies every other client, including a modified app.
Why: `request.auth` is the session. The path id has to equal `request.auth.uid`. A client-side `if (user)` check is not this rule.

**Case: create checks the document that is about to exist**

```text
match /orders/{orderId} {
  allow create: if request.auth != null
    && request.auth.uid == request.resource.data.userId
    && request.resource.data.status == "pending";
}
```

Result: the server accepts a create whose `userId` is the caller and whose `status` is `"pending"`. Any other payload is denied.
Why: on create there is no old document. `request.resource.data` is the document as it will be after this write.

**Case: read and update use the stored document**

```text
match /orders/{orderId} {
  allow read: if request.auth != null
    && request.auth.uid == resource.data.userId;
  allow update: if request.auth != null
    && request.auth.uid == resource.data.userId
    && request.resource.data.status == resource.data.status;
}
```

Result: the server allows the owner to read the order, and allows an update that leaves `status` as it is now.
Why: `resource.data` is the document as it is now. The update line compares that current status to the status in the incoming write.

**Case: a query that might include someone else's order**

```javascript
const q = query(collection(db, "orders"), where("status", "==", "pending"));
await getDocs(q);
```

Result: the SDK rejects the query when the rule allows only orders whose `userId` is the caller. The server does not return the caller's rows and hide the rest.
Why: rules are not filters. A query must be allowed for every document it might return. Constrain it with `where("userId", "==", uid)` so every possible row passes the rule.

**Case: an admin flag the user can write**

```text
match /users/{userId} {
  allow read, write: if request.auth != null && request.auth.uid == userId;
}
```

```javascript
await updateDoc(doc(db, "users", uid), { admin: true });
```

Result: the server allows the write. The user has granted themselves `admin: true`.
Why: that boolean lives on a document the same rule lets them edit. Admin checks use a document the user cannot edit, or a custom claim set from a server.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Hiding a button and skipping the rule | A modified client still calls the SDK | Put the same condition in rules |
| Querying a collection the rule only partly allows | The whole query is denied | Add a `where` that matches the rule |
| Reading `resource.data` on create | Create has no current document, so the check fails | Use `request.resource.data` for the new document |
| Storing `admin: true` on the user's own document | They can set it themselves | A document they cannot edit, or a custom claim |

### Tip

Write the query and the rule as a pair. If the rule looks at `userId`, the query needs the same constraint.

### Practice

Write a read rule so a signed-in user can read only their own orders, and say what a query of every pending order does under that rule.

### Solution

```text
match /orders/{orderId} {
  allow read: if request.auth != null
    && request.auth.uid == resource.data.userId;
}
```

A query of every pending order is rejected. Some of those documents belong to other users, and rules do not drop them. The query has to include `where("userId", "==", uid)`.

## 7. Messaging, storage, and hosting

### Concept

Cloud Messaging delivers a notification to a device or browser that opted in. The client cannot securely send an arbitrary notification to someone else. The send uses a server credential. Cloud Storage holds files: images, PDFs. A path such as `art/{uid}/{file}` plus a storage rule that requires `request.auth.uid == uid` is the same idea as a Firestore rule. The file is not a Firestore document. You store the download URL or the path on the document if the UI needs to find it. Hosting serves the built web app. A deploy uploads the `dist` or `build` folder. Hosting does not run your database. Firestore is already hosted.

### Terms

| Service | Stores |
| --- | --- |
| Firestore | Documents |
| Storage | Files |
| Auth | Accounts and sessions |
| Messaging | Push tokens and the delivery of notifications |
| Hosting | Static files and the built front end |

| Step | What happens |
| --- | --- |
| Permission | The browser asks the user |
| Token | The client receives a registration token and stores it for that user |
| Send | A server you trust, or a Cloud Function, sends to that token |
| Payload | A title, a body, and optional data fields |

### Methods

| Operation | Effect |
| --- | --- |
| Ask for notification permission | The browser prompts. No token exists if the person declines |
| Save the registration token | The client stores the token for that user so a server can address the device |
| Server send | A trusted server, using a server credential, delivers the title, body, and optional data |
| Storage upload to `art/{uid}/{file}` | The file lands in Cloud Storage. A rule can require `request.auth.uid == uid` |
| Write the path or download URL onto a document | The UI can find the file later. The file itself is not a Firestore field |
| Hosting deploy | Uploads the `dist` or `build` folder. It does not deploy Firestore |

### Examples

**Case: opt in, then keep the token**

```javascript
const permission = await Notification.requestPermission();
if (permission === "granted") {
  const token = await getToken(messaging);
  await setDoc(doc(db, "users", uid), { fcmToken: token }, { merge: true });
}
```

Result: the browser asks the user. On grant, the client receives a registration token and stores it for that user.
Why: Messaging can address only a device that opted in. The token is how the server names that device.

**Case: the client does not send the notification**

```javascript
await setDoc(doc(db, "mail"), {
  to: "someone-else",
  title: "Hello",
});
```

Result: a document write is not a secure notification send. Another person's device does not get a push from this call.
Why: the client cannot securely send an arbitrary notification to someone else. The send uses a server credential on a server you trust, or a Cloud Function.

**Case: a file path that matches the user**

```text
match /art/{uid}/{file} {
  allow read, write: if request.auth != null && request.auth.uid == uid;
}
```

Result: Storage allows the signed-in user to read and write only under `art/{their uid}/`. Another uid in the path is denied.
Why: the storage rule is the same idea as a Firestore rule. The file is still a file, not a document.

**Case: the document remembers where the file is**

```javascript
await updateDoc(doc(db, "works", workId), {
  imagePath: `art/${uid}/${fileName}`,
});
```

Result: the SDK stores a string on the document. The bytes stay in Storage.
Why: you store the download URL or the path on the document if the UI needs to find the file. The file is not a Firestore document.

**Case: Hosting uploads the build**

```text
dist/
  index.html
  assets/
```

Result: a Hosting deploy serves those static files. Firestore documents are untouched and stay on Firestore.
Why: Hosting serves the built web app. It does not run your database. Firestore is already hosted.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Sending a push from the browser with no server credential | You cannot prove the sender, and the token of another user is not yours to use | Send from a server you trust |
| Putting image bytes in a Firestore field | The document holds a huge value, and the file rules do not apply | Upload to Storage and store the path |
| A storage path with no uid check | Any signed-in client can write any folder | `request.auth.uid == uid` on `art/{uid}/{file}` |
| Expecting a Hosting deploy to update documents | The site files change. The documents do not | Write documents with the Firestore SDK |

### Tip

When the UI needs a picture, store a path or a download URL on the document and keep the bytes in Storage.

### Practice

Name the five services and what each one stores. Then say who is allowed to send a notification.

### Solution

| Service | Stores |
| --- | --- |
| Firestore | Documents |
| Storage | Files |
| Auth | Accounts and sessions |
| Messaging | Push tokens and the delivery of notifications |
| Hosting | Static files and the built front end |

A server you trust sends the notification, using a server credential, to a token from a device that opted in.

## Drill

1. Draw `users/{uid}` and `orders/{orderId}` with four fields between them.
2. Write the `where` and `orderBy` for homes in one city, cheapest first, ten rows.
3. Say why a query is rejected when the rule would allow only some of its documents.

### Solution 1

```text
users/{uid}
  displayName
  role

orders/{orderId}
  userId
  status
```

The four fields are `displayName`, `role`, `userId`, and `status`. The profile id is the uid you chose. The order id can be an auto id from `addDoc`.

### Solution 2

```javascript
import { collection, limit, orderBy, query, where } from "firebase/firestore";

const homes = query(
  collection(db, "homes"),
  where("city", "==", "Irvine"),
  orderBy("price"),
  limit(10),
);
```

`orderBy("price")` is ascending, so the cheapest rows come first. `limit(10)` keeps ten.

### Solution 3

Rules are not filters, and a client check is not a rule. The server allows a query only when every document it might return is allowed. If the rule would allow only some of those documents, Firestore rejects the query. It does not return the allowed rows and drop the rest. Add a `where` that matches the rule, such as `where("userId", "==", uid)` when the rule requires `request.auth.uid == resource.data.userId`.
