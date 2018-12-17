## Have you noticed `this` in Javascript?

`this` is a quite interesting being in the JS world. It is simple and does what it says - when you call a function it tries to know who is `this` caller calling it? :grin:

Let's start with the layman terms

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

Let's give this chance to a our new Dog

```javascript
const PedigreeLoverDog = {
  eatFast: null
}

PedigreeLoverDog.eatFast = Dog.eat;

console.log(PedigreeLoverDog.eatFast()); // ​​​​​I am eating my undefined​​​​​
```

Well this doesn't look good :confused:

We have assigned our new PedigreeLoverDog the function from generic Dog itself. Why does it say undefined?

Isn't `this` suppose to ask it's environment and just return the value?

It would, except the functional approach is very different in implementation than how it is written.


```javascript
const eat = function() {
  return `I am eating my ${this.food}`;
}

const Dog = {
  food: 'Pedigree',
  eat: eat
}

console.log(Dog.food) // Pedigree
console.log(Dog.eat) // [λ: eat]​​​​​
console.log(Dog.eat()) // I am eating my Pedigree​​​​​
```

the 'eat' function is separated out but still is able to reference the property on the Dog.

The property 'eat' does not create a new function from the function 'eat', it is actually referencing it!

```javascript
console.log(eat === Dog.eat) // true
```

```javascript
const a = function() {}
const b = function() {}

console.log(a === b) // false
```

What is happening here?

Rule Of Thumb - `this` always `binds`(more on bind coming soon) to the left hand caller in Javascript.

so when you say 'Dog.eat' - the caller is 'Dog' and it has the property called 'food'.

```javascript
Dog.newEat = eat

console.log(Dog.newEat()) // ​​​​​I am eating my Pedigree​​​​​
```
## Function Binding

`bind` is what you use to override the functionality such that - instead of `this` your object gets assignment of the default caller of the function.

Note: `bind` is available on function and not the objects!

It expects an object to replace the implicit `this` calling.

### Imlicit Binding

In browser the default binding for `this` is the `window` object. Check the example below

```javascript
window.globalVar = 'I ama global';

var checkGlobal = function() {
  return this.globalVar
}

console.log(checkGlobal()) // I am global
```

### Explicit binding

using prev example:

```javascript
console.log(
  eat()
) // ​​​​​​​​​I am eating my undefined​​​​​

console.log(
  eat.bind( { food: 'new food!' } )()
) // ​​​​​​​​​I am eating my new food!​​​​​
```

#### Why you need to use bind when using React?

If you've been creating components in React along with attaching event listeners then you've probably come across this problem of binding the functions before it's references.

```javascript
<button type="button" onClick={this.handleClick}>Click Me</button>
```

When using above code it will fail to do the desired operation and it needs a bind. Familiar?

#### Why does this happen?

because classes use strict mode - where the value of `this` being set to `undefined`.

Please check the example image below


