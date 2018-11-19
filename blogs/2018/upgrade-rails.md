# Upgrading Rails… Phew 😌

Here is something you need to know

[![rails.org](https://cdn-images-1.medium.com/max/800/1*19M7M0x-ERGHdImze4BD2Q.png)](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html)

<div align='center'>
<small><a style='color:grey' href='https://guides.rubyonrails.org/upgrading_ruby_on_rails.html'>https://guides.rubyonrails.org/upgrading_ruby_on_rails.html</a></small>
</div>
<br/>

**Rails** guides provide us with a brief information on how to upgrade the rails app, but we need to know more before we start to smooth our development process.

_**Upgrading rails is a painful task in rails app development life-cycle, it will be more & more painful if you are on older rails (ex: 3.2.x)**_

To make our workflow smoother and less painful, We will explore some insights in this article. I hope we enjoy this journey together 🙂

### Let's start with cleaning up Gemfile

While upgrade we will use `bundle install` and `bundle update` many times. And while doing bundle install or bundle update one may get a lot of dependency errors.

Then as per the error, we update our Gemfile by writing version number after it & try again. To make it even simpler and to get rid of maintaining Gemfile.lock, we can clean our Gemfile as below (see fig 1)

<div align='center'>
<img src='https://cdn-images-1.medium.com/max/800/1*ZUHap5UrPasQFaT9A_n6Yg.jpeg'>
<small style='color:grey'>
fig 1: Before and After
</small>
</div>
<br>
If we have a Gemfile like one on the right side, we can do

```
microposts 📂 (master) 🎋 $ > Gemfile.lock   #empty Gemfile.lock
```

whenever we want, and we do this out of frustration with dependency errors.

### Speed up rails commands

When we run any rails command like `rake db:migrate` or `bundle exec rspec`, it will take time to boot up rails app and execute. This will be time consuming and make our development slower & boring 😶 .

To speed up commands, following gems will help us

1. [zeus](https://github.com/burke/zeus)

If we go through their installation we can see zues is built to run outside bundler, so we don't need to put zeus in our Gemfile. Just, Install zeus and go to your app root and run

```
# tab 1
microposts 📂 (master) 🎋 $ zeus init
   create zeus.json
   create custom_plan.rb
microposts 📂 (master) 🎋 $ zeus start
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

After we run the above commands, open a new tab in the console and try the Available commands and notice the command speed. you can test the speed with the following commands

```
# tab 2
microposts 📂 (master) 🎋 $ time rake db:migrate
real 0m2.680s
user 0m1.672s
sys  0m0.599s
microposts 📂 (master) 🎋 $ time zeus rake db:migrate
real 0m0.002s
user 0m0.001s
sys  0m0.002s
```

using zeus we can run rails commands much faster. And you can add your own commands using zeus.

2. spring

Rails app will have spring by default when we create an app. Let's see how spring helps us in speeding up rails commands.

To use spring we just need to run following commands,

```
microposts 📂 (master) 🎋 $ time rake db:reset

real 0m3.600s
user 0m1.674s
sys  0m0.526s
microposts 📂 (master) 🎋 $ time spring rake db:reset

real 0m1.204s
user 0m0.208s
sys  0m0.089s
```

As you can see using spring it takes 1 sec, while normal command takes 3 sec and it will grow as your app grows.

As compared to zeus, spring server starts in the background and the server starts when we run our first command with spring. you can check spring server using

```
microposts 📂 (master) 🎋 $ ps aux | grep spring
aravind .. spring app | microposts | started 4 mins ago | developm..
aravind .. spring server | microposts | started 4 mins ago
```

You can prefer whichever gem suits for the development.

### Upgrading rails app

Rails have breaking changes in versions 3.2.22 to 4.2.10 which makes upgrade painful. But, Rails 4.2.10 and Rails 5.2.1 seems similar to upgrade from 4 to 5 will be easier.

_Let's see the problems we face while upgrade from 3.2.22 to 4.2.10_

1. [jquery-rails](https://github.com/rails/jquery-rails) and the asset gems

If we have older **jquery-rails** of version 2.3.0 in our app and other gems(ex: _active_admin_) may require jquery-rails of version > 2.3.0 or 4 then to upgrade gems, assets shouldn't be a problem.

To avoid upgrading **jquery** or **jquery-rails gem** what we can do is we can extract the assets from **jquery-rails** gem and include it in our **application.js** or wherever it needs. Here we can see how to do it

First, we need to check the installed version of **jquery-rails** in our app

visit https://github.com/rails/jquery-rails and click on branches, then click on tags select 2.3.0, see fig 2

<div align='center'>
<img src='https://cdn-images-1.medium.com/max/800/1*khZfvDh6VtwXY4VVORannA.png'/>
<small style='color:grey'>fig 2: select 2.3.0 version tag</small>
</div>
<br/>

Now copy **vendor/assets/javascripts** files to our app's folder **assets/javascript/jquery-rails** and require all files in one file and name it **assets/javascripts/jquery-rails/all.js**, see fig 3 & 4

<div align='center'>
<img src='https://cdn-images-1.medium.com/max/800/1*KTsjEdwRYLObqxANw_P0LA.png'/>
<p>
<small style='color:grey'>
fig 3: copied files
</small>
</p>
<img src='https://cdn-images-1.medium.com/max/800/1*2Ys2AO9Qu1MEgUfYuSmEoA.png'>
<p><small style='color:grey'>fig 4: all.js</small></p>
</div>
<br/>

Now, we are good to remove **jquery-rails** gem. go to your Gemfile and remove **jquery-rails**.
open **application.js** and import the **jquery-rails/all.js** in place of jquery-rails . see fig 5

<div align='center'>
<img src='https://cdn-images-1.medium.com/max/800/1*p5oxdekfY1n-ffUGD0TvtQ.png'>
<p><small style='color:grey'>fig 5: application.js changes</small></p>
</div>
<br/>

2. Upgrade gems

The https://guides.rubyonrails.org/upgrading_ruby_on_rails.html will give us a full brief about how we can proceed with the upgrade and what are the major changes/features in rails 3, 4 or 5.

But, here we have a different approach in upgrading rails viz., instead of upgrading all the gems at once, we can upgrade one gem at a time which covers a specific module(ex: jobs, mailers, ..) of our application.

But, sometimes a gem needs another gem to be updated along with it due to dependency, in that case, we need to upgrade both.

Upgrade all your gems one by one in older rails version. Then upgrade rails, if it needs the ruby upgrade then do both at the same time. Repeat this until you reach the latest rails version.

For example, If we are on rails 3.2.22, then we upgrade all the gems one by one where gems support rails 3.2.22 and then we will proceed with upgrade rails to version 4.0. But, rails 4.0 requires ruby 1.9.3 and above. If our app is running on older rails version (< 1.9.3) then we need to upgrade ruby then rails or both.

If we follow the above approach and upgrade gem one at a time and push it to the production, then we may get fewer errors and end up as happy developer. If there are any bugs in production then it will be easy to handle.

### Summary
While upgrading rails I have enjoyed a lot, as we are getting on a newer version of rails, and with fewer production bugs. I have upgraded 2 rails apps from Rails 3 to Rails 5 and I am happy to say that we are on a Rails 5.2.1 😃

Let us know what you are up to on-rails upgrade and do drop a comment if you have any queries.
