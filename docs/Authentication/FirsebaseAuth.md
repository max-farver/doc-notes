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

## In Web App

```
npm install firebase
```

1. Create firebase.js in utils, root, etc (wherever you want it)
2. Paste this:

   ```javascript
   import firebase from 'firebase/app';
   import 'firebase/auth';
   import 'firebase/firestore';
   import { useState, useEffect } from 'react';

   const firebaseConfig = {
     // your config here
   };

   if (!firebase.apps.length) {
     firebase.initializeApp(firebaseConfig);
   }

   export const useAuth = () => {
     const [state, setState] = useState(() => {
       const user = firebase.auth().currentUser;
       return user;
     });
     function onChange(user) {
       setState(user);
     }

     useEffect(() => {
       // listen for auth state changes
       const unsubscribe = firebase.auth().onAuthStateChanged(onChange);
       // unsubscribe to the listener when unmounting
       return () => unsubscribe();
     }, []);

     return state;
   };

   export const auth = firebase.auth;
   ```

The above code initializes the firebase instance for your app and provides a useAuth() hook that allows you to easily get info about the currently logged in user.

```javascript
const { user } = useAuth();
```

**_Yes, it's that easy, don't question it_**
