# Firebase and Firestore

Course 11 of 15. Previous: [10-sql.md](10-sql.md). Next: [12-tailwind.md](12-tailwind.md).

Firebase is a set of hosted services: login, a document database, file storage, messaging, and hosting. Firestore is the document database. It is not SQL, and it is not file storage. Cloud Storage is the file service.

## Contents

1. [The project and the client](#1-the-project-and-the-client)
2. [Authentication](#2-authentication)
3. [Documents and collections](#3-documents-and-collections)
4. [Reads and queries](#4-reads-and-queries)
5. [Writes](#5-writes)
6. [Security rules](#6-security-rules)
7. [Messaging, storage, and hosting](#7-messaging-storage-and-hosting)

## 1. The project and the client

A Firebase project has a config object. The values in that object identify the project. They are not a secret in the way a service-account private key is. Security rules are the lock.

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

| Name | Role |
| --- | --- |
| Web config | Safe to ship in a web app, because rules restrict it |
| Service account | A private key for servers and admin scripts. Never in the browser |
| Emulator | A local Auth and Firestore. Use it for tests |

## 2. Authentication

```javascript
import {
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  signOut,
  onAuthStateChanged,
} from "firebase/auth";

await createUserWithEmailAndPassword(auth, email, password);
await signInWithEmailAndPassword(auth, email, password);
await signOut(auth);

onAuthStateChanged(auth, (user) => {
  if (user) console.log(user.uid);
});
```

| Call | Result |
| --- | --- |
| `createUserWithEmailAndPassword` | A new user, and a session |
| `signInWithEmailAndPassword` | A session for an existing user |
| `signOut` | Clears the session on this device |
| `onAuthStateChanged` | Runs now and whenever the session changes |
| `user.uid` | The stable id. Use it as the document id for that person's profile |
| `sendPasswordResetEmail` | A reset email |

Other providers (Google, and so on) use `signInWithPopup` and a provider object. The session shape is the same: a user or null.

Failed calls reject the promise. Wrong password and unknown email are errors you show in the form. Do not treat them as a crash.

## 3. Documents and collections

Firestore stores documents inside collections. A document is a set of fields with an id. A collection is a list of documents. A document may contain a subcollection.

```text
users/{uid}
  displayName
  role

users/{uid}/carts/active
  items: [{ artworkId, qty }]

orders/{orderId}
  userId
  status
  total
```

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

Document ids you choose, such as the user id, make `users/{uid}` a direct read. Auto ids, from `addDoc`, are for records the user will have many of, such as orders.

Keep documents small. A growing list that can become huge belongs in a subcollection, not in one unbounded array.

## 4. Reads and queries

```javascript
import { doc, getDoc, collection, query, where, orderBy, limit, getDocs } from "firebase/firestore";

const snap = await getDoc(doc(db, "users", uid));
if (snap.exists()) {
  const profile = snap.data();
}

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

| Call | Result |
| --- | --- |
| `getDoc` | One document. `exists()` is false when the id is missing |
| `getDocs` | Every document in a query |
| `where` | A filter. `==`, `<`, `<=`, `>`, `>=`, `!=`, `in`, `array-contains` |
| `orderBy` | Sort. Inequality filters usually need the same field ordered first |
| `limit` | A maximum count |
| `startAfter` | The next page, using the last document of the previous page |

A compound query needs an index. The error message includes a link that creates it. Equality filters on several fields are fine. Range filters on two different fields are the case that needs planning.

`onSnapshot` listens. The callback runs with the current result and again when it changes. Unsubscribe when the component goes away.

```javascript
const stop = onSnapshot(doc(db, "orders", orderId), (snap) => {
  console.log(snap.data());
});
stop();
```

A missing index, a rule denial, and a missing document are three different outcomes. `exists() === false` is not a permission error. A permission error rejects the promise.

| | `getDoc` / `getDocs` | `onSnapshot` |
| --- | --- | --- |
| What you get | One promise of the current result | A callback now, and again on each change |
| Missing document | `exists()` is false. The promise still fulfills | The snapshot's `exists()` is false |
| Permission denied | The promise rejects | The error callback runs |
| How long it lasts | One read | Until you call the function it returned |
| When to use it | A page that can load once | A screen that must follow edits |

## 5. Writes

```javascript
import { setDoc, addDoc, updateDoc, deleteDoc, serverTimestamp } from "firebase/firestore";

await setDoc(doc(db, "users", uid), {
  displayName: "Henry",
  role: "artist",
  createdAt: serverTimestamp(),
});

await updateDoc(doc(db, "orders", orderId), {
  status: "approved",
});

await addDoc(collection(db, "orders"), {
  userId: uid,
  status: "pending",
  total: 40,
});

await deleteDoc(doc(db, "orders", orderId));
```

| | `setDoc` | `setDoc` with `{ merge: true }` | `updateDoc` | `addDoc` |
| --- | --- | --- | --- | --- |
| Who picks the id | You | You | You. The document must already exist | Firestore |
| Document missing | Creates it | Creates it | Fails | Creates a new one |
| Fields you do not mention | Deleted | Kept | Kept | There are no older fields |
| A whole replacement | Yes | No | No | It is a new document |

| Call | Effect |
| --- | --- |
| `deleteDoc` | Remove the document. Does not remove subcollections |
| `serverTimestamp()` | The server's clock, so clients cannot lie about the time |
| `arrayUnion` / `arrayRemove` | Add or remove an array element without rewriting the array |
| `increment(1)` | Change a number without reading it first |
| Write batch | Several writes that succeed or fail together |
| Transaction | Read, then write, and retry if the documents changed |

A batch cannot read. A transaction can. Use a transaction when the new value depends on the current value, such as "approve only if the status is still pending."

## 6. Security rules

Rules run on the server for every client request. They cannot be bypassed by a modified app. They are not filters. A query must be allowed for every document it might return. Firestore does not silently drop the forbidden ones.

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }

    match /orders/{orderId} {
      allow read: if request.auth != null
        && request.auth.uid == resource.data.userId;
      allow create: if request.auth != null
        && request.auth.uid == request.resource.data.userId
        && request.resource.data.status == "pending";
      allow update: if request.auth != null
        && request.auth.uid == resource.data.userId
        && request.resource.data.status == resource.data.status;
    }
  }
}
```

| Name | Meaning |
| --- | --- |
| `request.auth` | The signed-in user, or null |
| `request.auth.uid` | Their id |
| `request.resource.data` | The document as it will be after this write |
| `resource.data` | The document as it is now |
| `allow read` | Get and list |
| `allow write` | Create, update, and delete. Split them when the checks differ |

An admin flag has to live somewhere the user cannot grant themselves. A boolean `admin: true` on their own user document is not a lock, because the rule above would let them write it. Admin checks use a document the user cannot edit, or a custom claim set from a server.

Test rules with the emulator. A rule you have not tried will deny a legitimate write or allow a bad one.

## 7. Messaging, storage, and hosting

Cloud Messaging delivers a notification to a device or browser that opted in.

| Step | What happens |
| --- | --- |
| Permission | The browser asks the user |
| Token | The client receives a registration token and stores it for that user |
| Send | A server you trust, or a Cloud Function, sends to that token |
| Payload | A title, a body, and optional data fields |

The client cannot securely send an arbitrary notification to someone else. The send uses a server credential.

Cloud Storage holds files: images, PDFs. A path such as `art/{uid}/{file}` plus a storage rule that requires `request.auth.uid == uid` is the same idea as a Firestore rule. The file is not a Firestore document. You store the download URL or the path on the document if the UI needs to find it.

Hosting serves the built web app. A deploy uploads the `dist` or `build` folder. Hosting does not run your database. Firestore is already hosted.

| Service | Stores |
| --- | --- |
| Firestore | Documents |
| Storage | Files |
| Auth | Accounts and sessions |
| Messaging | Push tokens and the delivery of notifications |
| Hosting | Static files and the built front end |

## Drill

1. Draw `users/{uid}` and `orders/{orderId}` with four fields between them.
2. Write the `where` and `orderBy` for homes in one city, cheapest first, ten rows.
3. Say why a query is rejected when the rule would allow only some of its documents.
