---
name: useToggle
route: /react/hooks/usetoggle
menu: React - Hooks
---

# Simple Toggle

## Code

```javascript
import { useState } from 'react';
export const useToggle = initialValue => {
  const [isToggled, setToggled] = useState(initialValue);
  const toggle = () => setToggled(prev => !prev);

  return { isToggled, setToggled, toggle };
};
```

## Use

```javascript
const { isToggled, toggle } = useToggle(false);
```
