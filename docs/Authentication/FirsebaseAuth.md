---
name: Firebase
route: /authentication/firebase
menu: Authentication
---

# Firebase Auth (easiest)

- **_Note_** : This uses Google Auth, easy to change to others though once you get the idea.

## Firebase Console

1. Create new project
2. Select **Authentication** from the sidebar
3. Enable various authentications
4. Get project configuration details
   - Back to overview
   - Add a web app
   - Once done you should have the configuration details

## Client Side Auth

```
npm install firebase
```

1. Create firebase.js in utils, root, etc (wherever you want it)
2. Paste this:

   ```javascript
   import firebase from "firebase/app"
   import "firebase/auth"
   import "firebase/firestore"
   import { useState, useEffect } from "react"

   const firebaseConfig = {
     // your config here
   }

   if (!firebase.apps.length) {
     firebase.initializeApp(firebaseConfig)
   }

   export const useAuth = () => {
     const [state, setState] = useState(() => {
       const user = firebase.auth().currentUser
       return user
     })
     function onChange(user) {
       setState(user)
     }

     useEffect(() => {
       // listen for auth state changes
       const unsubscribe = firebase.auth().onAuthStateChanged(onChange)
       // unsubscribe to the listener when unmounting
       return () => unsubscribe()
     }, [])

     return state
   }

   export const auth = firebase.auth
   ```

The above code initializes the firebase instance for your app and provides a useAuth() hook that allows you to easily get info about the currently logged in user.

```javascript
const user = useAuth()
```

## Server Side Auth

### Admin setup

```
npm install firebase-admin
```

_In a separate file from client side_

```javascript
import * as admin from "firebase-admin"

if (!admin.apps.length) {
  admin.initializeApp({
    credential: admin.credential.cert({
      type: process.env.TYPE,
      project_id: process.env.PROJECT_ID,
      private_key_id: process.env.PRIVATE_KEY_ID,
      private_key: process.env.PRIVATE_KEY.replace(/\\n/g, "\n"),
      client_email: process.env.CLIENT_EMAIL,
      client_id: process.env.CLIENT_ID,
      auth_uri: process.env.AUTH_URI,
      token_uri: process.env.TOKEN_URI,
      auth_provider_x509_cert_url: process.env.AUTH_PROVIDER_CERT,
      client_x509_cert_url: process.env.CLIENT_CERT,
    }),
    databaseURL: /* dbUrl from firebase */,
  })
}

export const adminAuth = admin.auth
```

.env

```javascript
APP_URL=
TYPE=
PROJECT_ID=
PRIVATE_KEY_ID=
PRIVATE_KEY=
CLIENT_EMAIL=
CLIENT_ID=
AUTH_URI=
TOKEN_URI=
AUTH_PROVIDER_CERT=
CLIENT_CERT=
```

_Store all info from service account in environment variables_

### Api Routes

Login

```javascript
import { adminAuth } from "../../../utils/auth/firebaseAdmin"
import * as Cookies from "cookies"

export default async (req, res) => {
  const cookies = new Cookies(req, res)

  res.statusCode = 200 // Get the ID token passed and the CSRF token.
  const idToken = req.body.idToken.toString()
  // const csrfToken = req.body.csrfToken.toString()
  // // Guard against CSRF attacks.

  // if (csrfToken !== req.cookies.csrfToken) {
  //   res.status(401).send("UNAUTHORIZED REQUEST!")
  //   return
  // }

  // Set session expiration to 5 days.
  const expiresIn = 60 * 60 * 24 * 5 * 1000
  // Create the session cookie. This will also verify the ID token in the process.
  // The session cookie will have the same claims as the ID token.
  // To only allow session cookie setting on recent sign-in, auth_time in ID token
  // can be checked to ensure user was recently signed in before creating a session cookie.
  await adminAuth()
    .createSessionCookie(idToken, { expiresIn })
    .then(
      sessionCookie => {
        // Set cookie policy for session cookie.
        const options = {
          maxAge: expiresIn,
          httpOnly: true,
          secure: process.env.NODE_ENV === "PRODUCTION" ? true : false,
        }
        cookies.set("session", sessionCookie, options)

        return res.end(JSON.stringify({ status: "success" }))
      },
      error => {
        "Login Error \n", error
        return res.status(401)
      }
    )
}
```

Logout

```javascript
import { adminAuth } from "../../../utils/auth/firebaseAdmin"
import * as Cookies from "cookies"

export default async (req, res) => {
  const cookies = new Cookies(req, res)

  const sessionCookie = cookies.get("session") || ""

  cookies.set("session", "")
  await adminAuth()
    .verifySessionCookie(sessionCookie.toString())
    .then(decodedClaims => {
      return adminAuth().revokeRefreshTokens(decodedClaims.sub)
    })
  res.end("Logged Out")
}
```

Register

```javascript
import { adminAuth } from "../../../utils/auth/firebaseAdmin"
import * as Cookies from "cookies"

export default async (req, res) => {
  const cookies = new Cookies(req, res)

  const sessionCookie = cookies.get("session") || ""
  // Verify the session cookie. In this case an additional check is added to detect
  // if the user's Firebase session was revoked, user deleted/disabled, etc.
  await adminAuth()
    .verifySessionCookie(sessionCookie.toString(), true /** checkRevoked */)
    .then(decodedClaims => {
      res.end(
        JSON.stringify({
          message: "User is valid",
          isVerified: true,
          claims: JSON.stringify(decodedClaims),
        })
      )
    })
    .catch(error => {
      // Session cookie is unavailable or invalid. Force user to login.
      res.status(401).end(
        JSON.stringify({
          message: "UNAUTHORIZED",
          isVerified: false,
        })
      )
    })
}
```

### Client side of SS-auth

Same setup as before with client side, but no useAuth hook

_setUser comes from Context so the app can see who is logged in_

Login

```javascript
const onSubmit = async ({ email, password }) => {
  // When the user signs in with email and password.
  let hadError = false
  await auth()
    .signInWithEmailAndPassword(email, password)
    .catch(function(error) {
      alert("Invalid email / password")
      hadError = true
    })
  if (!hadError) {
    const idToken = await auth()
      .currentUser.getIdToken()
      .then(
        async idToken =>
          await fetch("/api/auth/login", {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({ idToken: idToken }),
          })
      )

    setUser(auth().currentUser.email)

    auth().signOut()
    router.push("/")
  }
}
```

Logout

```javascript
const logoutUser = async () => {
  await fetch("/api/auth/logout", {
    method: "POST",
  })
  setUser("")
  router.reload()
}
```

Register

```javascript
const onSubmit = async ({ email, password, confirm }) => {
  if (password === confirm) {
    let hadError = false
    await auth()
      .createUserWithEmailAndPassword(email, password)
      .catch(error => {
        hadError = true
      })
    if (!hadError) {
      const idToken = await auth()
        .currentUser.getIdToken()
        .then(
          async idToken =>
            await fetch("/api/auth/login", {
              method: "POST",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify({ idToken: idToken }),
            })
        )

      setUser(auth().currentUser.email)

      auth().signOut()
      router.push("/")
    }
  }
}
```

### NextJS

For pages that need to be hidden, check credentials before they load.

```javascript
export const getServerSideProps = async ctx => {
  await checkUserCredentials(ctx)
  return { props: {} }
}
```

checkUserCredentials file

```javascript
import fetch from "isomorphic-unfetch"

export const checkUserCredentials = async ctx => {
  const response = await fetch(`${process.env.APP_URL}/api/auth/verify`, {
    headers: {
      cookie: ctx.req.headers.cookie,
    },
  })
  if (response.status !== 200) {
    ctx.res.writeHead(302, { Location: "/login" }) // or wherever
    ctx.res.end()
  }
}
```

## TODO

- forgotten password
