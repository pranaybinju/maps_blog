# Showing Custom 404 error page for Netlify apps

We recently created a static app using Nextjs and Netlify. Everything was fine until we stumbled upon this error for invalid routes.

![Netlify 404 page =400x400](netlify-404.png)

This page indicates that we do not have any page defined for the route we entered. So as a fallback, Netlify redirects the user to its default `404.html` page.

## Overriding Netlify's default 404.html

So you might be wondering, if there was a way we could redirect user to our custom 404 error page? After reading [Netlify docs](https://www.netlify.com/docs/redirects/#custom-404), we see that this can be done by creating a custom 404.html page in our build directory. Once the file is found, Netlify will override its default 404.html page with this custom 404.html page that we just created. If we want to name this file something else then we will need to add following configuration in `netlify.toml` file(you can create one if you don't already have it and add following code in it), which should be present in your root directory.

```toml
[[redirects]]
  from = "/*"
  to = "/something.html"
  status = 404
```

Detailed explanation for `netlify.toml` file can be found [here](https://www.netlify.com/docs/netlify-toml-reference/). I'll brief them quickly here, so we understand underlying terminologies correctly.

- `[[redirect]]`: means we want to define redirect rule for our app, which controls how our pages are routed for certain rules. Rules are defined using various properties like `from`, `to`, `status`.

- `from`: In our code above, we want to redirect all invalid (not defined by us) routes to 404.html So `from` is the is that invalid route.

- `to`: This is error page that needs to be shown, it is by default set to `404.html` for status 404, we can override it by defining custom page here

- `status`: This is error status that need to be applied for above rule. We ask Netlify to show `to` html page for all `from` routes having error code as `404`.

## Redirecting to custom route for invalid routes

Since we are working with React, it would be good if we could use a component instead of html. Using component has many benefits like, we will be able to reuse any previously created components, like Header and Footer components. Also the anchor links will need to be hard-corded to make the html file work as Nextjs routes won't work with html file. So lets find a way to use components.

We can create `_error.js` which [Nextjs uses by default](https://nextjs.org/docs/#custom-error-handling) on server to render any invalid routes. This component will be rendered like any other component, so we will have our default headers and footers and also all the routing will remain intact as we can access Nextjs routing.

I have created a [Github repo](https://github.com/trojanh/nextjs-react-example) to demonstrate this component way of handling error so you can directly dive into the code and get some hands on.

If you check the code, our routes are as follows:

```js
module.exports = {
  "/": { page: "/" },
  "/about": { page: "/about" },
  "/async": { page: "/async" },
  "/hn": { page: "/hn" },
  "/error": { page: "/_error" }
};
```

We want to handle 404 error in such a way that it is consistent for local developments and also producton development environments. On local environment, error is handled by Nextjs(using `yarn dev`), and Nextjs uses `_error` page from `pages` folder to display error. You can read more about this [here](https://nextjs.org/docs/#custom-error-handling). Prod 404 error is handled by Netlify(using `yarn build` command), so we also want a route for redirecting user for error, this is can by done using `/error` route(you can keep this route as `_error` to match netlify page name, I find `/error` more readable).

Netlify configuration for this will look like:

```toml
[[redirects]]
  from = "/*"
  to = "/error"
  status = 404
```

Now everything looks good so far and we seemed to have solved our problem. You can preview the code in action till this point [here](https://5ca210ea90b5fe0008422eae--netlfiy-react-example.netlify.com/).There's one more thing left.

## Introducing a twist :beetle:

Now if we try any invalid route of the form `https://domain.netlify.com/error<any_text>` then we still see the Netlify `404.html` default page. Reason ?? seems its a bug. I reported this to the Netlify team (via email) and they accepted it was a bug. So how do we solve this bug?

This issue can be reproduced on this [netlify build](https://5ca210ea90b5fe0008422eae--netlfiy-react-example.netlify.com/errorsadasd) for the corresponding [git commit](https://github.com/trojanh/nextjs-react-example/commit/b2616add5474752fee233fb502231290bbfe104e)

I tried all possible combination to fix this issue from `netlify.toml` using `redirects` rule. I tried forcing all the pages to `/error` route(using `force=true`), but the problem seemed very stubborn. Refer [this link](https://www.netlify.com/docs/netlify-toml-reference/#redirects) for more on force=true.

After some research, I found a hack to fix this issue until Netlify fixes it from their end. Here's what I did,

`404.html`

```html
<!DOCTYPE html>
<html lang="en-US">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="refresh" content="0; url=https://netlfiy-react-example.netlify.com" />
    <script type="text/javascript">
      window.location.href = window.location.origin + "/_error";
    </script>
    <title>Page Redirection</title>
  </head>

  <body>
    If you are not redirected automatically, follow this
    <a href="https://netlfiy-react-example.netlify.com">Kiprosh</a>.
  </body>
</html>
```

For routes of the form `https://domain.netlify.com/error<any_text>`, this code snippet will override the default `404.html` of Netlify and once the user comes to this page, he will be redirected to our `/error` route page using browser location API.

Hope this helped you and saved your time finding fixes from Netlify which do not exist yet.

Thanks for reading and feel free drop any questions about this or even possible solutions which you think can still fix this.
