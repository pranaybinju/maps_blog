# Showing Custom 404 error page for Netlify apps

We recently created a static app using Nextjs and Netlify using React. Everything was fine until we discovered this error for invalid routes

---link to image

This page indicates that we do not have any route for the invalid route we entered so as fallback, Netlify redirects the user to its default `404.html` page.

So you might be wondering, if there was way we could redirect user to our custom error page?
After reading [Netlify docs](https://www.netlify.com/docs/redirects/#custom-404), we could do it by

```
/* /404.html 404
```

This should be saved in file called `_redirect` in your root path of the project. This will tell Netlify that we want the users to see `404.html`. This configuration is provided by default and need not be written(the above 404 page we saw in screenshot is Netlify's `404.html` and is rendered automatically). We can simply create a `404.html` and Netlify is smart enough to indentify and replace it with default `404.html`

But since we are working with React, its not very convinient and right way to use 404.html. Firstly, we won't be able to reuse any component here, so all the headers and footer will need to be re-written here. Also the anchor links will need to be hardcorded to make the html file work. So this won't be a very good way of showing that page.

Instead what we can do is we can create a new route and component for 404 error page. We can create `_error.js` which [Nextjs uses by default](https://nextjs.org/docs/#custom-error-handling) on server to render any invalid routes. This component will be rendered like any other component, so we will have our default headers and footers.

Also, I prefer using [`netlify.toml`](https://www.netlify.com/docs/netlify-toml-reference/) file over `_redirect` as we can provide all our Netlify related configurations right inside this file. So the redirect configuration now becomes a bit different for `netlify.toml` file like below:

```
[[redirects]]
  from = "/*"
  to = "/404.html"
  status = 404
```
As I mentioned earlier this configuration is provided by Netlify and need not be written again but we want to use it for our `_error` route, so we want to redirect user to





