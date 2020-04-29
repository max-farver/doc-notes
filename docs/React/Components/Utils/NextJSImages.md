---
name: NextJS Images
route: /react/next-optimized-images
menu: React - Components - Utils
---

# Optimized Images in NextJS

This is a guide to optimizing images in a similar fashion to gatsby-image.

```
npm install next-compose-plugins next-optimized-images webp-loader lqip-loader
```

_Option: install `image-trace-loader` instead of `lqip-loader` if you want to trace iamges instead of blur._

next.config.js

```javascript
const withPlugins = require("next-compose-plugins")
const optimizedImages = require("next-optimized-images")
const path = require("path")

module.exports = withPlugins(
  [
    [
      optimizedImages,
      {
        /* config for next-optimized-images */
        // ########## trace config #############
        imageTrace: {
          color: "#fff",
        },
        // #####################################
      },
    ],
    // your other plugins here
  ],
  {
    webpack(config) {
      config.resolve.alias.images = path.join(__dirname, "images")
      return config
    },
  }
)
```

ImageComponent

```javascript
const Image = ({ src, innerStyle }) => {
  return (
    <picture className={`relative ${innerStyle}`}>
      <img
        className={`absolute w-full top-0 left-0`}
        src={require(`../../media/images/${src}?lqip`)}
      />
      // ######## trace version #########
      <img src={require(`images/${src}?trace`).trace} />
      // ################################
      <source
        className={`absolute w-full top-0 left-0`}
        srcset={require(`../../media/images/${src}?webp`)}
        type="image/webp"
      />
      <source
        className={`absolute w-full top-0 left-0`}
        srcset={require(`../../media/images/${src}`)}
        type="image/jpeg" // edit this as needed, could get it from src
      />
      <img
        className={`absolute w-full top-0 left-0`}
        src={require(`../../media/images/${src}`)}
      />
    </picture>
  )
}

export default FSImage
```

innerStyle is for height/width/rounded/etc (is this doesn't work, wrap it in a div, I haven't tested it with just `<picture>`)
