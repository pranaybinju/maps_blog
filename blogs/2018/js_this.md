##### Have you noticed ***this*** in JavaScript?
> `this` is quite interesting thing(reserved keyword) in the JS world. It is simple and does what it says - when you call a function it tries to know who is `this` trying to call me? 😬


Here's an easy and simple example to start with 👇🏼

```javascript
const Dog = {
  food: 'Pedigree',
  eat: function() {
    return `I am eating my ${this.food}`;
  }
}

console.log(Dog.food) // Pedigree
console.log(Dog.eat) // [λ: eat]​​​​​
console.log(Dog.eat()) // I am eating my Pedigree​​​​​
```

Great! now let's use this `eat` to for new custom Dog 🐶

As `eat` is defined in the `Dog` object, it knows how to eat food.

Next we'll provide this functionality to new Object PedigreeLoverDog 🐕

```javascript
const PedigreeLoverDog = {
  eat: null
}

PedigreeLoverDog.eat = Dog.eat;

console.log(PedigreeLoverDog.eat()); // I am eating my undefined
```

🤔 `this` doesn't look good here - (pun intended 😬)

A misconception some one here can have with `PedigreeLoverDog.eat = Dog.eat` is PedigreeLoverDog will have the access to `food` via `Dog`. But that's not the case.
🙇

 **Let's play around the concept here** 👨🏻‍💻


```javascript
const eat = function() {
  return `I am eating my ${this.food}`;
}

const Dog = {
  food: 'Pedigree',
  eat: eat
}

console.log(Dog.food) // Pedigree
console.log(Dog.eat) // [λ: eat]
console.log(Dog.eat()) // I am eating my Pedigree
```

> Note: The 'eat' function is separated out but still is able to find the `food` on the Dog.

By default `this` will `bind` to the left hand caller. Doing `Dog.eat` makes `eat` ask for food from `Dog`.


```javascript
console.log(eat === Dog.eat) // true
console.log(eat()); //I am eating my undefined
```


The left hand object is the scope of `this` for a function , this is the reason when `eat` is called without `Dog`, it loses the context.

##### More on **bind**ing

`bind` is what you use to provide scope to the `this` keyword. When a function is referenced inside the Object - it has default binding to the Object also known as Implicit Binding. This explains how inside `Dog#eat` gets to know `food` via `this`.

Note: `bind` is available on function and not the objects!


### Implicit Binding

In browser the default binding for `this` is the `window` object. Check the example below

```javascript
window.globalVar = 'I am a global';

var checkGlobal = function() {
  return this.globalVar
}

console.log(checkGlobal()) // I am a global
```

-------

### Explicit Binding

using previous example:

```javascript
console.log(eat()) // I am eating my undefined

console.log(eat.bind( { food: 'new food!' } )()) // I am eating my new food!
```

-----

##### **bind** with React

If you've been creating components in React and attached event listeners then you've probably come across this situation of using `bind` to the functions to pass the scope.

```jsx
<button type="button" onClick={this.myHandler}>Hello</button>
```
> When using above code it will fail to do the desired operation and it needs a bind. Familiar?


###### Yes! but why we need to?
> blame `ES6` and not `React`.

ES6 introduced `class`. And now you can structure Components as Classes.

But a class uses `strict mode` and that sets the default value of  `this` to `undefined` instead of `window` or `global`.

When using React components with ES6 classes - make use of binding to provide `this` the context and not leaving it undefined.

Checkout behaviour - "this" defaults to global but becomes `undefined` with "use strict" 👇
<p><img src="https://drive.google.com/uc?authuser=0&id=1cJPkT_9-4ogTq6QlfCFPTOeJKaLR-0ti&export=download" style="max-width: 850px;"></p>

##### Arrow functions `=>`

One of the approach to solve such issues is the usage of Arrow Functions introduced in ES6. These functions have a default binding to `this`.

```jsx
class FooBar extends React.Component{
  myHandler = () => {
    console.log(this); 
  }

  render() {
    return(
      <button type="button" onClick={this.myHandler}>Hello</button>
    )
  }
}
```

##### Summary

- It does not matter if you declare the functions inside an object or outside of it. It's the scope of `this` that matters.

- Object calling the function is the default binding for a `this`. Also called as `implicit` binding.

- If the function reference is assigned to variable then `implicit` binding will default to `global` in node & `window` in browser environment.

- To change the scope of this, use `function.bind(object)` also called as `explicit` binding

- ES6 classes work with `"use strict"` that prevents binding of `this` to the default scope.

- Make use of the Arrow functions to reduce verbosity of `explicit` binding!

--------

Reach out to us and let me know your thoughts on `this` 😁

- Karan Valecha [@IamKaranV](https://twitter.com/iamkaranv?lang=en)
