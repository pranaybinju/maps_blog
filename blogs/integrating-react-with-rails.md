# Gracefully integrate Reactjs with Ruby on Rails in SaaS Applications

Ruby on Rails has been one of the most versatile and most widely used technology for creating a fast-paced application. One can also say that its an extremely productive technology stack that enables quick development with easy deployment. All these characteristics make Ruby on Rails a near perfect choice of framework for businesses & startups.

On the frontend side of things, building snappy & reactive UIs can be challenging as DOM manipulation using plain-vanilla JavaScript and jQuery can be a snag at times. Keeping track of multiple elements, dealing with multiple UI states, updating UI based on user interaction while keeping business logic in check is a tough problem to deal with. Another major concern is an actual refresh/reload of the page while navigating from one page to another, this induces the assets to be loaded again and again which makes the webpage slow and non-interactive.

Frontend frameworks are great when it comes to solving these above-mentioned problems. They provide robust abstraction & intuitive APIs to work with while providing a great developer experience.

## React - For Fast UI

[React](https://reactjs.org/) is a JavaScript framework (developed by Facebook) which is most widely used for creating interactive UIs. Its declarative views make the code predictable and easier to debug. React enforces component-based structure of building UIs. Component logic is written in JavaScript instead of templates, you can easily pass rich data/properties through your app and keep the state out of the DOM. React creates a virtual DOM, which helps in fast rendering and handling the state changes with ease without interfering with the actual DOM.

It can be considered as the "V" of the ["MVC"](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) architecture. We can pass properties to the component, as soon as the property gets changed, the component automatically gets rerendered, which results in improving the performance of the page by reducing the API calls.

In the first part of this article, we will discuss how to integrate Reactjs with Ruby on Rails application and in the second part, we will discuss common challenges we might face during integration.

**Part-I: Integrating Reactjs with Ruby on Rails**

React can be integrated with Ruby on Rails in many ways, but we suggest official [react-rails](https://github.com/reactjs/react-rails) gem which is the simplest way to get started with React and JSX in a Rails application.

Some of the features of using `react-rails` gem are:

It automatically renders React server-side and client-side
* Supports Webpacker 3.x, 2.x, 1.1+
* Supports Sprockets 4.x, 3.x, 2.x
* Lets you use JSX, ES6, TypeScript, CoffeeScript

> This gem supports rails version greater than 4.2 because webpacker gem does not support rails versions lower than 4.2. In order to make it work with rails versions lower than 4.2, you have an option of either using an asset pipeline or have to do some workaround on setting webpack. 

Before moving on let's learn about [Webpack](https://webpack.js.org/) in brief.

Webpack is a popular JavaScript bundler built on top of Node.js. Some of the advantages of webpack are: 
* Faster build time
* Modular javascript bundles
* Cross-Browser compatibility
* Auto polyfill for targetted browsers

So to begin with let's create our rails app (assuming you already have rails framework installed on your PC)

```bash
rails new react-rails-app
cd react-rails-app
```

We can now start by adding the gem to our dependencies.

```ruby
#Gemfile
.
.
gem 'webpacker'
gem 'react-rails'
.
```
Install the added gem, by running

```bash
bundle install
```

Run the generator

```bash
rails webpacker:install
rails webpacker:install:react
rails generate react:install
```

This gives us a components directory under `app/assets/javascripts/`, a manifest file and adds them to our `application.js`'s dependency.

(https://blog.kiprosh.com/content/images/2018/12/Screen-Shot-2018-12-22-at-5.50.46-PM.png)

```javascript
directory for your React components
app/javascript/components/ 

ReactRailsUJS setup in 
app/javascript/packs/application.js

for server-side rendering
app/javascript/packs/server_rendering.js 
```

Now let's link the Javascript pack in Rails view using javascript_pack_tag helper:

```ruby
<!-- application.html.erb in Head tag below turbolinks -->
<%= javascript_pack_tag 'application' %>
```

Let's create our first component by running the following command in our console
 
```bash
rails g react:component HelloWorld greeting:string
```

This will generate a component in `app/javascript/components/` directory which will look somewhat like this

```javascript
app/javascript/components/HelloWorld.js

import React from "react"
import PropTypes from "prop-types"
class HelloWorld extends React.Component {
  render () {
    return (
      <React.Fragment>
        <h1>Greeting: {this.props.greeting}</h1>
      </React.Fragment>
    );
  }
}

HelloWorld.propTypes = {
  greeting: PropTypes.string
};
export default HelloWorld
```

To view, let's create a controller with action

```bash
rails g controller greetings hello
```

This creates a controller with action hello, a template `hello.erb` in `views/greetings` and a get route for this action

Now in `hello.erb` we'll call the react component and pass the props

```ruby
<%= react_component("HelloWorld", { greeting: "Hello from react-rails." }) %>
```

Let's view all the work we have done in the browser

```bash
rails server
```

Visit `localhost:3000/greetings/hello`, you'll see the output from the component we created above on your browser window:

https://blog.kiprosh.com/content/images/2018/12/Screen-Shot-2018-12-24-at-10.48.21-AM.png
https://blog.kiprosh.com/content/images/2018/12/Screen-Shot-2018-12-23-at-10.10.39-AM.png

**Part-II: Common challenges we might face while integrating Reactjs in Ruby on Rails applications**

Now let us look at the challenges that we might face while integrating React with Ruby on Rails web applications:

**Issue 1: react-rails gem's incompatibility issues with the older versions of Rails**

There are some compatibility issues with `react-rails` gem when we try to integrate React with Rails versions lesser than 4 because the webpacker gem is not supported by rails versions lesser than 4.

As we integrated React to Rails 3, we also faced that issue. So, we decided to use asset-pipelining as react-rails gem provides a pre-bundled React.js & a UJS driver to the Rails' asset pipeline. 

Get started by adding the react-rails gem:

```ruby
#Gemfile
.
.
gem 'react-rails'
.
.
```

And then install the react generator:

```bash
rails g react:install
```

Then restart your development server.
This will add:
> some `//= requires` to application.js

```javascript
app/assets/javascript/application.js

//= require axios.min
//= require react
//= require react_ujs
//= require components
```
> `components/` directory for React components
> `server_rendering.js` for server-side rendering 

Now, you can create React components in .jsx files:

```javascript 
app/assets/javascripts/components/helloWorld.jsx


window.HelloWorld = createReactClass(
  { render() { 
     return (
      <React.Fragment>
        <h1>Greeting: {this.props.greeting}</h1>
      </React.Fragment>
    );
   } 
 });

// or, equivalent:

class HelloWorld extends React.Component {
  render() { 
   return (
      <React.Fragment>
        <h1>Greeting: {this.props.greeting}</h1>
      </React.Fragment>
    );
  }
}
```

Then, you can render those components in views:

```ruby 
<%= react_component("HelloWorld", {greeting: "Hello World"}) %>
```

We will see a similar result on the browser as we saw in Part-I.

**Issue 2: ES6, ES7 syntax incompatibilities with Rails versions lesser than 3**

While integrating React with Rails 3 we faced some issues in compiling the ES6, ES7 syntaxes. `react-rails` uses a transformer class to transform JSX in the asset pipeline. The transformer is initialized once, at boot. 

We can provide a custom transformer to `config.react.jsx_transformer_class`. The transformer must implement `#initialize(options)`, where options is the value passed to config.react.jsx_transform_options. In order to use ES6/ES7 syntaxes we need to add following setting in the `config/application.rb`: 

```ruby 
 config.react.jsx_transform_options = {
   optional: ['es7.classProperties']
 }
```

Restart the rails server and things will work as expected.

**Issue 3: Issues with using pre-render in Rails versions lesser than 3**

> What is Pre-rendering? 

> Prerendering is a process to preload all elements on the page in preparation for a web crawler to see it. 

Pre-rendering your React application is useful if you want to increase the SEO performance of your website and make it visible to search engines. Most of the search engine robots do not execute client side javascript code. That makes the websites which are made by React invisible to search engines. `react-rails` gem provides an option to use `pre-render` for pre-rendering the application.

```ruby
<%= react_component('HelloWorld', {greeting: 'Hey John'}, { prerender: true }) %>
```

However, when we tried pre-rendering by adding the pre-render option, components were rendering properly but they were not responding to the actions performed on them.

So, we used the `ReactRailsUJS`'s `mount` and `unmount` methods and also added the generated `server_rendering.js` in the `config/initializers/react_server_rendering.rb` file to fix this issue:

```ruby 
config/initializers/react_server_rendering.rb

# To render react components in production, precompile the server rendering manifest:

Rails.application.config.assets.precompile += ['server_rendering.js']
```

Now add these methods to the js.erb files to render the components:

```ruby
# Here are some options for rendering the components

// Mount all components on the page:
ReactRailsUJS.mountComponents()
// Mount components within a selector:
ReactRailsUJS.mountComponents(".my-class")
// Mount components within a specific node:
ReactRailsUJS.mountComponents(specificDOMnode)

// Unmounting works the same way:
ReactRailsUJS.unmountComponents()
ReactRailsUJS.unmountComponents(".my-class")
ReactRailsUJS.unmountComponents(specificDOMnode)
```

There is still more to it. For more details on this gem, you can visit the official documentation for this gem, and look out various options here(https://github.com/reactjs/react-rails).

Do write to me if you find this interesting or if you have any suggestions or doubts. Also, I’d like to grow my readership. So please share this blog with your friends/peers if you found this blog helpful.
