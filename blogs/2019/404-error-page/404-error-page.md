# Showing Custom 404 error page for Netlify apps

We recently created a static app using Nextjs and Netlify using React. Everything was fine until we discovered this error for invalid routes

---link to image

This page indicates that we do not have any route for the invalid route we entered so as fallback, Netlify redirects the user to its default `404.html` page.

So you might be wondering, if there was way we could redirect user to our custom error page?
After reading [Netlify docs](https://www.netlify.com/docs/redirects/#custom-404), we could do it by

```
/* /404.html 404
```

This should be saved in file called `_redirect` in your root path of the project. This will tell Netlify that we want the users to see `404.html`. This configuration is provided by default and need not be written(the above 404 page we saw in screenshot is Netlify's `404.html` and is rendered automatically). We can simply create a `404.html` and Netlify is smart enough to identify and replace it with default `404.html`

But since we are working with React, its not very convenient and right way to use 404.html. Firstly, we won't be able to reuse any component here, so all the headers and footer will need to be re-written here. Also the anchor links will need to be hardcorded to make the html file work. So this won't be a very good way of showing that page.

Instead what we can do is we can create a new route and component for 404 error page. We can create `_error.js` which [Nextjs uses by default](https://nextjs.org/docs/#custom-error-handling) on server to render any invalid routes. This component will be rendered like any other component, so we will have our default headers and footers.

Also, I prefer using [`netlify.toml`](https://www.netlify.com/docs/netlify-toml-reference/) file over `_redirect` as we can provide all our Netlify related configurations right inside this file. So the redirect configuration now becomes a bit different for `netlify.toml` file like below:

```
[[redirects]]
  from = "/*"
  to = "/404.html"
  status = 404
```
As I mentioned earlier this configuration is provided by Netlify and need not be written again but we want to use it for a custom `_error` route. Whenever a page is not found, we should show user this `_error` component. Using this, our error page will feel part of our app and will let users navigate to other pages right from this page.

[This repo](https://github.com/trojanh/next-pwa-boilerplate) on github has all the above code with Netlify url as https://demo-netlify-reactjs.netlify.com/,

If you check the code I have created routes as below
```
module.exports = {
  '/': { page: '/' },
  '/about': { page: '/about' },
  '/async': { page: '/async' },
  '/hn': { page: '/hn' },
  '/error': { page: '/_error' }
}
```

Here I want my error page(i.e `_error` component) to reside on `/error` route, so I map
`'/error': { page: '/_error' }`. The reason why I have named the component as `_error` is because Nextjs uses `_error` component to handle error by default. We also want to show this error page on netlify error for routes.

Now everything looks good so far and we seemed to have solved our problem. There's one more thing left.

Now if we try any route of the form `https://demo-netlify-reactjs.netlify.com/error<any_text>` then we still see the Netlify `404.html` default page. Reason ?? seems its a bug. I reported this to the Netlify team and they accepted it was a bug. So how do we solve this bug?









