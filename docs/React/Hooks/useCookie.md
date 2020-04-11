---
name: useCookie
route: /react/hooks/usecookie
menu: React - Hooks
---

# Easy way to work with cookies

## Code

```javascript
import { useState, useEffect } from 'react';
import Cookies from 'js-cookie';

export const useCookie = key => {
  const initial = Cookies.get(key);
  const [cookie, setStateCookie] = useState(initial);

  useEffect(() => {
    Cookies.set(key, cookie);
  }, [cookie, key]);

  return [cookie, setStateCookie];
};
```

## Use

```javascript
const [testCookie, setTestCookie] = useCookie('key of cookie');
```
