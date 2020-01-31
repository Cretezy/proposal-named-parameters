# Named Parameters

Stage 0

This proposal builds upon the previous [object rest/spread](https://github.com/tc39/proposal-object-rest-spread) proposal.

## Motivation

Functions often accept optional parameters with defaults, passed in as a destructured object.

Declaring named parameters without the object syntax hopes to be a simpler way of using optional parameters.

This proposal also adds Optional Destructuring for simplify common use of named parameters.

### Example

```js
function logTime(time = new Date(), {{ asIso8601 = false }}) {
  if (asIso8601) {
    console.log(time.toISOString()); // '2019-03-13T21:53:33.493Z'
  } else {
    console.log(time.toString()); // 'Wed Mar 13 2019 17:53:33 GMT-0400 (Eastern Daylight Time)'
  }
}

logTime(new Date("2019-03-13T21:53:33.493Z"), asIso8601: true);
```

JavaScript programmers can currently accomplish the same thing through this syntax:

```js
function logTime(time = new Date(), { asIso8601 = false } = {}) {
  if (asIso8601) {
    console.log(time.toISOString()); // '2019-03-13T21:53:33.493Z'
  } else {
    console.log(time.toString()); // 'Wed Mar 13 2019 17:53:33 GMT-0400 (Eastern Daylight Time)'
  }
}

logTime(new Date("2019-03-13T21:53:33.493Z"), { asIso8601: true });
```

A (slightly) more complex example:

```js
function resizeImage(image, {{ height, width, quality = 85, ...rest }}) {
  console.log(`${height}x${width}`); // '300x400'
  console.log(`${rest.format}@${quality}`); // 'webm@85'
}

resizeImage(image,
  height: 300,
  width: 400,
  format: "webm"
);
```

An example with no destructuring:

```js
function setup(config) {
  console.log(config.useColor); // true
}

setup(useColor: true);
```

An example with no positional arguments:

```js
function setup({{ useColor }}) {
  console.log(useColor); // true
}

setup(useColor: true);
```

## Semantics

### Named parameters

Named parameters are only allowed after all positional parameters. They are passed as the last argument of the function, as an object.

The following example is valid syntax:

```js
logTime(asIso8601: true);
```

The following example is not valid syntax:

```js
logTime(asIso8601: true, new Date());
```

When not passing all positional arguments to a function, all remaining parameters should be considered as `undefined` (and use default values). Example:

```js
function copyFile(from = "from.txt", to = "to.txt", {{ overrideIfExist = true }}) {
  console.log(`Copying ${from} -> ${to}`);
  // ...
}

copyFile("from2.txt", overrideIfExist: true); // 'Copying from2.txt -> to.txt'
```

Since all arguments are optional in the above example, using named parameters solves this problem nicely:

```js
function copyFile({{ from = "from.txt", to = "to.txt", overrideIfExist = true }}) {
  console.log(`Copying ${from} -> ${to}`);
  // ...
}

copyFile(from: "from2.txt", overrideIfExist: true); // 'Copying from2.txt -> to.txt'
```

### Optional Destructuring

The `{{ ... }}` syntax is equivalent to `{ ... } = {}`.

This ensures using destructuring will happen if no named parameters are passed (prevents destructing on `null`).

## Inspiration

This proposal is mostly inspired by [Dart's optional parameters](https://www.dartlang.org/guides/language/language-tour#optional-parameters):

```dart
void enableFlags({bool bold, bool hidden, int size = 12}) {
  // ...
}

enableFlags(bold: true, hidden: false);
```

[C#'s named arguments](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments) is another similar feature.


```csharp
void PrintOrderDetails(string sellerName, int orderNum, string productName)
{
    // ...
}

PrintOrderDetails("Gift Shop", 31, "Red Mug");
PrintOrderDetails(sellerName: "Gift Shop", orderNum: 31, productName: "Red Mug");
```

## Other Considerations

- This syntax does not appear to be interfering with type checkers (TypeScript/Flow). To be clarified with their respective teams about possible collisions.
