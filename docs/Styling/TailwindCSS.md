---
name: Tailwind CSS
route: /styling/tailwind
menu: Styling
---

# Tailwind CSS

Utility CSS framework to speed up development, this is how you set it up in different projects.

```
npm install tailwindcss
npm install --save-dev stylint-config-recommended
```

stylint is more of a nice to have for vscode purposes

### Main CSS File

```css
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";
```

### Create Tailwind Config File

```
npx tailwindcss init
```

**Don't use this file, use output.css (shown below how to make)**

```
npx tailwindcss build <CSSFile> -o <OutputFile>
```

_This will need to be run when you make changes to it_

Then you're good to go.

_See below for different platforms_

## NextJS

```
npm install @fullhuman/postcss-purgecss
npm install --save-dev autoprefixer postcss-preset-env
```

Create a file postcss.config.js

```javascript
const purgecss = [
  "@fullhuman/postcss-purgecss",
  {
    content: ["./components/**/*.js", "./pages/**/*.js"],
    defaultExtractor: content => content.match(/[\w-/.:]+(?<!:)/g) || [],
  },
]
module.exports = {
  plugins: [
    "postcss-import",
    "tailwindcss",
    "autoprefixer",
    ...(process.env.NODE_ENV === "production" ? [purgecss] : []),
  ],
}
```

Replace build script in package.json with

```
"build": "next build && npx tailwindcss build styles/globalStyles.css -o styles/output.css",

```

Import your output css file in '\_document.js' to use it throughout the project.

_Could possibly use original css file when in development environment for speed purposes_
