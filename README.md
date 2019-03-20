# Cerealizer

Why yes, it's another serialization library for JavaScript, because this problem that damn well ought to be solved as a built-in feature of the runtime environment still isn't. This is in spite of JavaScript runtimes having to include not one but two serialization mechanisms as built-ins: the well-known `JSON.stringify()`, and the much less well-known HTML Structured Clone algorithm that's used to pass object graphs between Workers.

The inadequacies of JSON are well-known: it doesn't even support all the basic built-in JavaScript classes (RegExp, Date, the TypedArray family, Error, and not to mention the new BigInt, Map, Set, and Symbol), it cannot handle arbitrary object graphs (only trees), it does not preserve class identity, it does not preserve functions or object properties, it does not properly preserve special values like undefined and NaN. It supports only a very limited subset of JavaScript's data description, and can serialize that. Worse, while for some JSON-able objects it generates errors (cycles, for example, generate exceptions) others, such as Maps, just get silently swallowed.

The HTML Structured Clone algorithm improves somewhat on JSON. It can handle graphs that contain cycles, for example, and it supports some more data types. But it retains other flaws: no functions or class identities, for example. It's also awkward to use; there's no direct API for it. Instead, the environment just calls it automagically when passing data to a Worker or back.

A search of npm finds a reasonable assortion of serialization libraries. But none of them seem to be ideal, exactly. They all retain one or more of the above flaws, rendering them undesirable for the role I have. JavaScript includes some very subtle ways of determining an object's type (`Object.prototype.toString` accessing the inaccessible `[[Class]]` property is particularly awesome, for example), and they require some finesse to replicate. I want to support classes but without requiring a no-args constructor to give me an empty object to populate. Object graphs need to be preserved, including cycles. I want to support the new datatypes where it makes sense to (`WeakMap` and `WeakSet` don't make sense to deserialize), and properly handle extensions of any built-in types (not just `Object`!).

As with any JavaScript serialization library, there's some things that can't be serialized: closures and the forthcoming private properties. TC39 is opting for hard privacy over deep cloning, so you'll need to write custom serialization code to handle classes that have true private data, whether that privacy is using the classic closure approach, or the new `#` prefix on property names. And indeed there are hooks provided for this very scenario.

I've only written and tested on Node 11.11, which is current at the time of writing. I'm using TypeScript, compiled down to es2017 mode, because in one spot I need to capture the type of a native `async` function. Aside from that one spot, 2016 mode is likely to be fine. There's preliminary support for decorators, too, using the `Reflect` metadata API.
