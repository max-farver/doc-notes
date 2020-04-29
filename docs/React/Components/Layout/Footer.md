---
name: Footer
route: /react/footer
menu: React - Components - Layout
---

# Footer Placement

Nothing special, this is just for making sure the footer stays at the bottom of the page.

## General Idea

Make the wrapper around the whole app have a min-height of 100vh with display-flex and flex-col then either justify-between or put margin-top on the footer equal auto (this one is for if the header is not fixed for some reason, and it allows you to not wrap the rest of the app again).

## NextJS

Use style tag in the head of \_document.

\_document.js

```javascript
import React from "react"
import NextDocument, { Head, Main, NextScript } from "next/document"

export default class Document extends NextDocument {
  render() {
    return (
      <html lang="en">
        <Head>
          <style>
            {`
            #__next {
              display: flex;
              flex-direction: column;
              min-height: 100vh;
              justify-content: space-between;
            }
            `}
          </style>
        </Head>
        <body>
          <Main />
          <NextScript />
        </body>
      </html>
    )
  }
}
```
