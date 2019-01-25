## Config

https://docs.ghost.org/concepts/config/

```
"mail": {
    "transport": "SMTP",
    "options": {
        "service": "Mailgun",
        "auth": {
            "user": "postmaster@example.mailgun.org",
            "pass": "1234567890"
        }
    }
}
```

### Amazon SES

```
"mail": {
    "transport": "SMTP",
    "options": {
        "host": "YOUR-SES-SERVER-NAME",
        "port": 465,
        "service": "SES",
        "auth": {
            "user": "YOUR-SES-ACCESS-KEY-ID",
            "pass": "YOUR-SES-SECRET-ACCESS-KEY"
        }
    }
}

```

From address
By default the 'from' address for mail sent from Ghost is set to the title of your publication, for example <ghost@your-publication.com>. To override this to something different, use:


```
"mail": {
    "from": "myemail@address.com",
}
```

A custom name can also be provided:


```
"mail": {
    "from": "'Custom Name' <myemail@address.com>",
}
```

Admin URL
Admin can be used to specify a different protocol for your admin panel or a different hostname (domain name). It can't affect the path at which the admin panel is served (this is always /ghost/).


```
"admin": {
  "url": "http://example.com"
}
```



https://github.com/TryGhost/Ghost/issues/5675






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




## Adding a "Subscribe to Tag" RSS feed button

https://grantwinney.com/5-quick-hacks-for-your-ghost-theme/

Ghost can generate RSS feeds for individual tags, but they're not all that discoverable. To help your visitors, add the last line in the following snippet to the ghost/content/themes/casper/tag.hbs file. It combines your blog url and the tag url to generate a custom RSS link your visitors can use.

```
<h1 class="page-title">{{name}}</h1>
<h2 class="page-description">
    {{#if description}}
        {{description}}
    {{else}}
        A {{../pagination.total}}-post collection
    {{/if}}
</h2>
<a class="icon-feed rss-tag" href="{{@blog.url}}{{url}}rss/">Subscribe to this tag</a>


```
This produces a new "subscribe" link under the tag description when viewing a particular tag. (Here I've applied some color and shadow effects to the other elements too.)



## Adding thumbnail images next to post listings

Many of the themes in another popular blogging platform ;) load a "featured" thumbnail image next to each individual post when viewing lists of posts. If, like me, you miss this feature and think the front page is a bit bland without thumbnails, you can modify one of the templates to include them.

Add the second line in the snippet below to ghost/content/themes/casper/partials/loop.hbs. That's the file that handles lists of posts, like on the front page of your blog or when you're viewing an individual tag. It inserts your post's cover image if there is one.

```
<section class="post-excerpt">
    {{#if image}}<img src="{{image}}" class="front-page-image" />{{/if}}
    <p>{{excerpt words="26"}} <a class="read-more" href="{{url}}">&raquo;</a></p>
</section>

```

The front-page-image class is for adding styling, because it's gonna look pretty ugly without it. Here's how I defined it. You could place the following in a separate file and reference it in the `{{!-- Styles'n'Scripts --}}` section of default.hbs, include it in the "Code Injection" part of the admin panel, or just place it inline in the img element. Play around with it until you get something you like.


```
/* Add thumbnail image to the main posts list */
.front-page-image {
    width: 200px;
    max-height: 200px;
    float: right;
    margin-left: 30px;
    margin-bottom: 15px;
    box-shadow: 5px 5px 3px gray;
}

```


Here's what it ends up looking like. Notice how I have longer descriptions too. You can change {{excerpt words="26"}} to a larger number in the tag.hbs file. I set it to 100.

![](https://grantwinney.com/content/images/2017/03/ghost-thumbnails-in-post-list.png)




## Using a post's meta data as its title and excerpt in post listings


Ghost has built SEO functionality right into its core. While editing a post, you can add meta data about it. Ghost then uses that meta data to generate markup that can be consumed by search engines or used when posting to social media sites.

![](https://grantwinney.com/content/images/2017/03/ghost-meta-data-in-post-list-3.png)

What if you wanted to use this data, when present, as the "title" and "excerpt" on your blog in lists of posts like on the front page? I mean, you've already taken the extra step of adding a concise description of your post, so why not use that instead of having Ghost simply display the first xx words of your post?

Here's where ghost/content/themes/casper/partials/loop.hbs loops through each of your posts, writing the title and excerpt.


```
{{#foreach posts}}
<article class="{{post_class}}">
    <header class="post-header">
        <h2 class="post-title"><a href="{{url}}">{{title}}</a></h2>
    </header>
    <section class="post-excerpt">
        <p>{{excerpt words="26"}} <a class="read-more" href="{{url}}">&raquo;</a></p>
    </section>
```

You can modify that to check for the presence of a meta_title and meta_description first, then fall back to the regular title and first xx words of your post if needed.

```
{{#foreach posts}}
<article class="{{post_class}}">
    <header class="post-header">
        <h2 class="post-title"><a href="{{url}}">{{#if meta_title}}{{meta_title}}{{else}}{{title}}{{/if}}</a></h2>
    </header>
    <section class="post-excerpt">
        <p>{{#if meta_description}}{{meta_description}}{{else}}{{excerpt words="26"}}{{/if}} <a class="read-more" href="{{url}}">&raquo;</a></p>
    </section>
```

And here's how it renders. The first post has a meta title and meta description specified. The second post has only a meta description, and the third has neither.


![](https://grantwinney.com/content/images/2017/03/ghost-meta-data-in-post-list-4.png)



---

https://grantwinney.com/creating-a-table-of-contents-for-your-blog/

https://github.com/grantwinney/table-of-contents-for-html-page



---


## [How to find all the posts with a specific tag in Ghost and iterate over them?](https://stackoverflow.com/questions/30362782/how-to-find-all-the-posts-with-a-specific-tag-in-ghost-and-iterate-over-them)

13

In case anyone comes across this still, this is now possible. Here is how you can do it via the get helper:

```
{{#get "posts" filter="tags:tagname"}}
    {{#foreach posts}}
         <p>{{title}}</p>
    {{/foreach}}
{{/get}}

{{#get "posts" filter="tags:[tag1, tag2]"}}
    {{#foreach posts}}
         <p>{{title}}</p>
    {{/foreach}}
{{/get}}
```



https://stackoverflow.com/questions/23640333/is-there-a-way-to-give-ghost-static-pages-access-to-the-posts-variable-that-in



[Ghost: newest post with specific tag on the front page](https://stackoverflow.com/questions/27356986/ghost-newest-post-with-specific-tag-on-the-front-page)


[API Reference / Handlebars](https://docs.ghost.org/api/handlebars-themes/helpers/has/)
