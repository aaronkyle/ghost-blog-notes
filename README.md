

## Adjusting Image Sizes In Ghost


`/var/www/ghost/content/themes/css/assets/`

`screen.css`



## Responsive Images

https://docs.ghost.org/api/handlebars-themes/responsive-images/

Ghost's dynamic image sizes feature 

Configuration

First, in your theme's `package.json` - you'll need to set up which sizes you'd like to use. You can change these sizes at any time and Ghost will automatically regenerate copies of your images at the specified sizes, so treat this more like a cache than anything else. Generally speaking, less is better. Try to not have more than 10 image sizes so your media storage doesn't grow out of control.

Here's a sample of the [image sizes in Ghost's default Casper theme](https://github.com/TryGhost/Casper/blob/master/package.json).

```
"config": {
    "image_sizes": {
        "xxs": {
            "width": 30
        },
        "xs": {
            "width": 100
        },
        "s": {
            "width": 300
        },
        "m": {
            "width": 600
        },
        "l": {
            "width": 1000
        },
        "xl": {
            "width": 2000
        }
    }
}

```


Using image sizes

Once your image sizes are defined, you can pass a size parameter to the `{{img_url}}` helper in your theme to output an image at a particular size.

`<img src="{{img_url feature_image size="s"}}">`

If you want to build out full responsive images, then create your html srcsets passing in multiple image sizes, and let the browser do the rest.

Here's an example from Ghost default Casper theme implementation `index.hbs`:


```
<img class="post-image"
    srcset="{{img_url feature_image size="s"}} 300w,
            {{img_url feature_image size="m"}} 600w,
            {{img_url feature_image size="l"}} 1000w,
            {{img_url feature_image size="xl"}} 2000w"
    sizes="(max-width: 1000px) 400px, 700px"
    src="{{img_url feature_image size="m"}}"
    alt="{{title}}"
/>

```



Compatibility

Ghost image sizes will be automatically generated for all images uploaded directly to Ghost, and will regenerated as needed automatically whenever you change an image, a list of sizes, or the theme being used. Unlike other platforms, there's no manual work needed to manage image sizes, it's all done in the background for you.

**Dynamic image sizes are not compatible with externally hosted images.** If you insert images from Unsplash or you store your image files on a third party storage adapter then the image url returned will be determined by the external source.
