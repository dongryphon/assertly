# Assert

The `Assert` class is the primary export of **assertly**.

This means an easy way to use the class is:

    const expect = require('assertly').expect;

`Assert` instances created behind the scenes to support statements like this:

    expect(x).to.not.be(2);

The two values above are described as the "actual" ("x") and "expected" (2) values.

Understanding these mechanical pieces is not typically important until `Assert` is
[extended](./Extensibility.md) or [integrated](./Integration.md).

## The Assert Class

An instance of this class is created by the `expect` static method:

    class Assert {
        static expect (value) {
            return new this(value);  // use "this" to support derived classes
        }

        constructor (value) {
            this.value = value;

            // ...
        }

        // ...
    }

Now consider the dot-chain in the statement from above:

    expect(x).to.not.be(2);

The `expect(x)` method creates a `new Assert(x)` and returns that instance. Then
the rest of the dot-chain runs against it. First are the two "modifiers" (`to` and
`not`) and then the "assertion" (`be`).

Each piece of the dot-chain (`to`, `not` and `be`) are first processed as a property
getter and are tracked in a `_modifiers` object.

Note: Due to this use of property getters means **assertly** cannot support IE8 even
if transpiled.

Breaking down the above:

    let a = expect(x);  // same as "a = new Assert(x)"

    a.to;       // sets "a._modifiers.to = true" and returns "a"
    a.not;      // sets "a._modifiers.not = true" and returns "a"
    a.be(2);    // sets "a._modifiers.be = true" and calls "a.be()"

So in the end we have this:

    a._modifiers = { to: true, not: true, be: true };

The assertion is also collected in the `_modifiers` object because the property getter
cannot know if `a.be` will be invoked or use as a step in a deeper dot-chain:

    expect(x).to.not.be.within(5, 10);

## Conjunctions

Sometimes a value needs to be used in multiple assertions. This can be explicit:

    expect(x).to.not.be(2);
    expect(x).to.not.be.within(5, 10);

Or combined using `and`:

    expect(x).to.not.be(2).and.not.be.within(5, 10);

This is handled by the assertion method returning a helper object that only provides
the `and` method (as well as a `then` method when using promises). The `and` property
holds a new `Assert` instance with the same "actual" value as the original `Assert`.
The previous `Assert` instance is stored as `_previous` on the new instance.

## Custom Modifiers and Assertions

All of the modifiers and assertions provided by `Assert` are dynamically added
using the `register()` method.

For more on these use cases, see [Extensibility](./Extensibility.md).
