# Upgrading Ruby on Rails insights

For the past few months at Kiprosh, we've done versions upgrade of multiple Ruby on Rails mid/large scale applications. One of them was running rails version 3.2.22. That's where we find out that, we need to make lot of changes in our codebase to run our application on version 4.0. Hence, we thought to write a blog to share what we have learned so far.


![upgrade ruby on rails](https://blog.kiprosh.com/content/images/2018/11/upgrade_rails_d.png)
[upgrading ruby on rails](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html)

**Rails** guides provide us with a brief information on how to upgrade the rails app, but we need to know more before we start to smooth our development process.


Upgrading rails is a painful task in rails app development life-cycle, it will be more & more painful if we are on older rails version(ex: 3.2.x)


To make our workflow smoother and less painful, we will explore some insights in this article. I hope we enjoy this journey together 🙂

### Let's start with cleaning up Gemfile

While upgrade we will use bundle install and bundle update many times. When we run bundle install or bundle update we may get gems dependency error(s).

As per the dependency error(s), we update our Gemfile by writing version number after it and bundle again. If that not solves our problem then it is better to bundle gems again with empty Gemfile.lock rather than resolving those dependency error(s). To empty Gemfile.lock, we need to maintain versions in Gemfile as below (see fig 1)

![fig 1: Before and After Gemfile](https://blog.kiprosh.com/content/images/2018/11/gemfile.jpeg)
fig 1: Gemfile

If we have a Gemfile like one on the right side (fig 1), we can empty Gemfile.lock if we are in a situation where to resolve dependencies will be more hectic than bundling gems.

Hence, we can run following command whenever we need or tired of resolving dependencies.

```
$ > Gemfile.lock  # this will empty Gemfile.lock
```

> Note: check gemspec of each gem to find out which gem makes the dependency error(s)/issue(s).

### Speed up rails commands

As our application grows in terms of features and codebase, rails commands will become slower (may take 15-25 sec for larger apps) and development will become slow & boring.

The following gems will help us to speed up rails commands

### [1. zeus](https://github.com/burke/zeus)

Here we will see how zeus helps us to speed up commands.

As compared to other gems, zeus is designed to work outside bundler(no need to add in Gemfile) and compatible with [parallel_tests](https://github.com/grosser/parallel_tests) where spring lacks native support for gem [parallel_tests](https://github.com/grosser/parallel_tests).

Let us see how zeus works,

```
# tab 1

rails_app $ gem install zeus
Successfully installed zeus-0.15.14
Parsing documentation for zeus-0.15.14
Installing ri documentation for zeus-0.15.14
Done installing documentation for zeus after 1 seconds
1 gem installed

rails_app $ zeus init
   create zeus.json
   create custom_plan.rb

rails_app $ zeus start
Starting Zeus server v0.15.14
[ready] [crashed] [running] [connecting] [waiting]
boot
  └── default_bundle
      ├── development_environment
      │   └── prerake
      └── test_environment
          └── test_helper
Available Commands: [waiting] [crashed] [ready]
zeus generate (alias: g)
zeus destroy (alias: d)
zeus dbconsole
zeus rake
zeus runner (alias: r)
zeus console (alias: c)
zeus server (alias: s)
zeus test (alias: rspec, testrb) [run to see backtrace]
```

After we run the above commands, open a new tab in the console and try the available zeus commands and notice the command speed. you can test the speed of commands as below

```
# tab 2

rails_app $ time rake db:migrate

real 0m2.680s
user 0m1.672s
sys  0m0.599s

rails_app $ time zeus rake db:migrate

real 0m0.002s
user 0m0.001s
sys  0m0.002s
```

we recommend to stay on zeus for rails version 3 & 4. zeus does not support rails 5 as hot reload mechanism changed in rails 5.1.

> zeus is written in [golang](https://golang.org/)


### [2. spring](https://github.com/rails/spring)

spring gem developed in pure ruby and that's the only reason to add it in rails 4.1 stack as a default gem under development group. It will take time when we run command for the first time in the console then a background process will start for spring server.

we need to prepend commands with spring, to use spring for speed up rails commands.

```
rails_app $ time rake db:reset

real 0m3.600s
user 0m1.674s
sys  0m0.526s

rails_app $ time spring rake db:reset

real 0m1.204s
user 0m0.208s
sys  0m0.089s
```

As compared to zeus, spring server starts in the background and the server starts when we run our first command with spring. you can check spring server using the following command

```
rails_app $ ps aux | grep spring
aravind .. spring app | microposts | started 4 mins ago | developm..
aravind .. spring server | microposts | started 4 mins ago
```

Since, spring doesn't natively support multi-thread, it won't be a best match for gem parallel_tests. For more info you can check this [issue](https://github.com/grosser/parallel_tests/issues/286#issuecomment-31909827).

> Note: zeus works faster as compared to spring.

### [3. bootsnap](https://github.com/Shopify/bootsnap)

Once we reach rails version 5.0, we can use bootsnap instead of zeus, as zeus won't support in rails 5.1.

Rails added bootsnap(v1.1.0) as a default gem in version 5.2 (see fig. 2)

![bootsnap](https://blog.kiprosh.com/content/images/2018/11/bootsnap.png)
fig 2: bootsnap in rails 5.2

To have bootsnap in our app, first we need to add bootsnap gem in our Gemfile

```
# Gemfile

...

gem 'bootsnap', '1.1.0', require: false

group :development, :test do
...


```

Now, add bootsnap in `config/boot.rb`

```
# config/boot.rb

ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../Gemfile', __dir__)

require 'bundler/setup' # Set up gems listed in the Gemfile.
require 'bootsnap/setup' # Speed up boot time

...

```

We recommend to use bootsnap, once we are on rails version 5 or later.

### Upgrading rails app

After following above steps, we are good to go for upgrading rails.

As we are working on rails upgrade, we have upgraded a rails app which is running on a rails version 3.x. Here, we want to share the breaking changes in versions 3 and 4 which makes the upgrading process painful and hectic.

The initial breaking change we encountered with rails 4 was routes. In rails 4 match has been removed and we have to use one of  get , put, post and delete methods for HTTP GET, PATCH, POST and DELETE respectively.

Replacing `match`  with the respective http method is time consuming if the routes are not well documented.

we can find complete list of changes in [upgrade ruby on rails guides](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html#upgrading-from-rails-3-2-to-rails-4-0).

Let's continue with the problems we have faced while upgrading rails from version 3.x to 4.x.

### Removing [jquery-rails](https://github.com/rails/jquery-rails)

If we have older jquery-rails of version 2.3.0 in our app and other gems(ex: active_admin) may require jquery-rails of version 3.0 or later, then to upgrade all other gems, assets gems shouldn't create a problem as we don't want to upgrade jquery.

> Note: Rails 5.1 dropped jquery from default rails stack. Find more info [here](https://github.com/rails/rails/issues/25208).

To avoid upgrading jquery or jquery-rails gem what we can do is we can extract the assets from jquery-rails gem and include it in our application.js or wherever it needs. Lets see how we did it.

First, we need to check the installed version of jquery-rails in our app

visit [jquery-rails](https://github.com/rails/jquery-rails) and click on branches, then click on tags then select 2.3.0, see fig 3

![jquery-rails](https://blog.kiprosh.com/content/images/2018/11/jquery-rails-v2_3_0.png)
fig 3: checking out jquery-rails version 3

Now copy vendor/assets/javascripts files to our app's folder assets/javascript/jquery-rails and require all files in one file and name it assets/javascripts/jquery-rails/all.js, see fig 4 & 5.

![fig 5: app/assets/javascripts/all.js](https://blog.kiprosh.com/content/images/2018/11/copied-files.png)

fig 5: app/assets/javascripts/all.js

Now, we can remove jquery-rails gem. go to your Gemfile and remove jquery-rails. open application.js and import the jquery-rails/all.js in place of jquery-rails . see fig 6

![fig 6: application.js diff after removing jquery-rails gem ](https://blog.kiprosh.com/content/images/2018/11/diff.png)

fig 6: application.js diff after removing jquery-rails gem

Similarly, we can remove other assets gems if they have a dependency problem with other gems.

### Upgrading gems

The [upgrading ruby on rails at rails.org](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html) page will give us a full brief about how we can proceed with the upgrade and what are the major changes/features in rails 3, 4 or 5.

But, here we have a different approach in upgrading rails viz., instead of upgrading all the gems at once, we can upgrade one gem at a time which covers a specific module(ex: jobs, mailers, ..) of our application.

Upgrade all your gems one by one in older rails version. Then upgrade rails, if it needs the ruby upgrade then do both at the same time. Repeat this until you reach the latest rails version.

For example, If we are on rails 3.2.22, then we upgrade all the gems one by one where gems support rails 3.2.22 & 4.0 and then we will proceed with upgrade rails to version 4.0. But, rails 4.0 requires ruby 1.9.3 and above. If our app is running on older ruby version (< 1.9.3) then we need to upgrade ruby then rails or both.

If we follow the above approach and upgrade gem one at a time and push it to the production, then we may get fewer production bugs/errors and end up as happy developer. If there are any bugs in production then it will be easy to handle.

### Summary

While upgrading rails I have enjoyed a lot, as we are getting on a newer version of rails, and with fewer production bugs. I have upgraded 2 rails apps from Rails 3 to Rails 5 and I am happy to say that we are on a Rails 5.2.1 😃

Let us know what you are up to on rails upgrade and do drop a comment if you have any queries/thoughts.
