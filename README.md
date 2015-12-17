# JavaScript Enumerations

Below are some thoughts on how they should/might work in JavaScript. Some of the ideas below use Rust and Swift as prior art.

## Declearing an Enumeration

```js
enum Planet { Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune }
```

By default, the first option should have a `rawValue` of 0. The second should have a `rawValue` of 1 and so on. We can override, the `rawValue` by providing a default argument.

```js
enum Planet { Mercury = 0, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune }
```

If a default argument is provided to the first value of the enumeration is an integer, then subsequent values with count up from there. For example, if `Mercury` has a default value of 1, then `Venus` should have a `rawValue` of 2.

### Further Questions

What should happen if an intermediate value has a `rawValue` provided? For example:

```js
enum Planet { Mercury, Venus, Earth = 3, Mars, Jupiter, Saturn, Uranus, Neptune }
```

What should happen if the first value is provided a `rawValue` that is not an integer?

```js
enum Planet { Mercury = "mercury", Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune }
```

Finally, what should happen if an intermediate value has a `rawValue` which is a non-integer?

```js
enum Planet { Mercury, Venus, Earth = "earth", Mars, Jupiter, Saturn, Uranus, Neptune }
```

## Instantiating an Enumeration

So, we can declare an enumeration, but how would we instantiate one?

```js
enum Planet { Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune }

let planet = Planet.Mercury();
let anotherPlanet = Planet(0);
```

In Swift, we'd just say:

```swift
enum Planet { Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune }

let planet = Planet.Mercury;
```

In Rust, we'd do something similar:

```rust
enum Planet {
  Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}

fn main() {
  let planet: Planet = Planet::Mercury;
}
```

But, this feels a little out of place to how we tend to instantiate objects in JavaScript.

```js
enum Planet {
  Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}

let planet = Planet.Mercury();
```

We can't just return a symbol from this constructor because symbols aren't objects and we can't affix any methods to them. This will eventually limit the API at our disposal (as I found out the hard way in the control flow section).

```js
enum Planet {
  Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}

let planet = Planet.Mercury();

planet.value // Symbol(mercury)
planet.rawValue // 0
```

We can also get a reference to the unique symbol behind all of the properties on the enumeration itself.

```js
enum Planet {
  Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}

Planet.Mercury.value; // Symbol(mercury)

let planet = Planet.Mercury();

planet.value === Planet.Mercury.value; // true
```

I don't love the `Planet.Mercury.value` syntax, but it allows us the flexibility to compare the symbols but also have methods like `rawValue` and whatnot. This will be useful later when we talk about control flow with enumarations. It also allows us the opportunity to seperate referencing a enumeration from instatiating an enumeration.

#### Changing the Value of an Instatiated Enumeration

Rust and Swift are statically typed, which means that once an enumeration is stored in a variable, it should only be able to change to another value of the same enumeration. In Swift, we have a syntax like this:

```swift
enum Planet {
  Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}

var planet = Planet.Venus

planet = .Earth
```

JavaScript, being dynamically typed, probably doesn't need a syntax like this. Instead, we might do something like this:

```js
enum Planet {
  Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}

let planet = Planet.Venus();

planet = Planet.Earth();
```

### Control Flow Based on Enumerations

One of the primary use cases for Enumerations is control flow. In Swift, we'd something like

```swift
let planet = Planet.Mercury

switch planet {
case .Mercury:
    print("A bit to close to the sun for my taste.")
case .Venus:
    print("A toxic ball of gas.")
case .Earth:
    print("I live here!")
// etc…
}
```

```rust
let planet = Planet::Mercury;

match planet {
  Planet::Mercury   => println!("A bit to close to the sun for my taste."),
  Planet::Venus  => println!("A toxic ball of gas"),
  Planet::Earth => println!("I live here!")
  // etc…
}
```

So, what could something similar look like in JavaScript?

```js
enum Planet {
  Mercury = 0, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}

let planet = Planet.Mercury();

if (planet.Mercury) {
  console.log('This planet is Mercury!');
}

if (planet.rawValue === 0) {
  console.log('This planet is Mercury!');
}

switch (planet.value) {
  case Planet.Mercury.value:
    console.log('A bit to close to the sun for my taste.');
    break;
  case Planet.Venus.value:
    console.log('A toxic ball of gas.');
    break;
  case Planet.Earth.value:
    console.log('I live here!');
    break;
  // etc…
}
```

Ideally, a instantiated enumeration would have a set of properties that would hold boolean values based on it's state.

```js
enum Planet {
  Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}

let planet = Planet.Mercury();

planet.mercury; // true
planet.venus; // false
planet.earth; // false
planet.mars; // false
planet.jupiter; // false
planet.saturn; // false
planet.uranus; // false
planet.neptune; // false
```
