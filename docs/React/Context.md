---
name: Context
route: /react/context
menu: React
---

# Context for Global State

## Create

```javascript
import React, { createContext, useContext, useState, useEffect } from 'react';
import { useCookie } from './hooks'; // see useCookie in 'React - Hooks'
import { useAuth } from './firebase';

// set defaults
export const AppContext = createContext({
  user: null,
});

export const PageWrapper = ({ children }) => {
  // get data from cookie if it is available
  const [userCookie, setUserCookie] = useCookie('user');
  const [user, setUser] = useState(userCookie);

  // updates the state cookie to keep data after refresh or between sessions
  useEffect(() => {
    setUserCookie(user);
  }, [user]);

  return (
    <AppContext.Provider
      value={{
        user: user,
        setUser: setUser,
      }}
    >
      {children}
    </AppContext.Provider>
  );
};

export const useAppState = () => {
  return useContext(AppContext);
};
```

## Use

```javascript
const { user } = useAppState();
```
