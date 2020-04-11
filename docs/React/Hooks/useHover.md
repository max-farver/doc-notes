---
name: useHover
route: /react/hooks/usehover
menu: React - Hooks
---

# Change state when hovering over an element or component

## Code

```javascript
import { useState, useMemo } from 'react';

export const useHover = () => {
  const [isHovered, setIsHovered] = useState(false);
  const bind = useMemo(() => {
    return {
      onMouseOver: () => setIsHovered(true),
      onMouseLeave: () => setIsHovered(false),
    };
  }, []);

  return [isHovered, bind];
};
```

## Use

```javascript
const [isHovered, bindHover] = useHover()


<Card {...bindHover} style={{ background: isHovered ? 'blue' : 'red' }}>
  // inner stuff
</Card>
```
