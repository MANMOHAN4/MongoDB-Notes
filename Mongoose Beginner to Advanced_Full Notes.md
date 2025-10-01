<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# This is the transcript of professor's lectures video. It is basically a notes. Can you format it properly in readable format. Don't condense the content. Let the terms used by prof be as it is and let it be descriptive and easy to understand. Don't miss anything.

I want to study as a documentation. So include each and everything in the transcript as documentation and give. I don't want to miss anything from professor's notes.

Do for this also as you did earlier.

Here is a documentation‑style Markdown for the attached transcript, preserving all terminology, examples, and explanations without condensing any content, so it can be studied as reference notes end‑to‑end.[^1]

# Mongoose Beginner to Advanced: Full Notes


***

### Overview

Knowing the basics of mongoose is great for building out projects, but understanding advanced concepts will take code to the next level, so this guide covers not only the basics but also important advanced concepts that are often missing from other tutorials. Mongoose is a wrapper around MongoDB, so having a solid grasp of MongoDB is recommended; the lecture references a full MongoDB crash course and a free MongoDB cheat sheet that lists every command, noting that these commands are available through mongoose as well. The workflow starts by setting up a blank project, installing dependencies, connecting to a local database, and then moving through schemas, models, queries, validation, query helpers, population, virtuals, and middleware in depth.[^1]

***

### Project setup

- Create a blank project and add a file named `script.js` (the filename can be anything) to run mongoose code interactively during development.[^1]
- Initialize npm with `npm init -y`, and if npm is not installed, install Node.js first to get npm as part of the toolchain.[^1]
- Install mongoose using `npm i mongoose`, and install `nodemon` as a dev dependency with `npm i -D nodemon` to auto‑reload the script on change.[^1]
- Update `package.json` replacing the default test script with a dev script, for example: `"dev:start": "nodemon script.js"`, then run with `npm run dev:start` to execute `script.js` and auto‑restart on save; a quick test like `console.log('hi')` will show live reloads.[^1]

```json
// package.json (scripts snippet)
{
  "scripts": {
    "dev:start": "nodemon script.js"
  }
}
```


***

### Connecting to MongoDB

- Require mongoose at the top of the script and call `mongoose.connect('mongodb://localhost/testdb')` where `testdb` is the desired database name; on save, the process connects even if no immediate output appears.[^1]
- `mongoose.connect` can take success and error callbacks; logging “connected” in the success callback verifies connection, and logging the error in the error callback surfaces connection failures.[^1]
- Mongoose will queue database operations while connecting, so interaction can begin without awaiting the connection explicitly, which is why a simple `mongoose.connect` is sufficient for most cases.[^1]

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost/testdb',
  () => console.log('connected'),
  (err) => console.log(err)
);
```


***

### Core concepts

- A **schema** defines the structure of data, e.g., a user schema with name, email, age, birthday, etc., and it is the blueprint of document shape and constraints.[^1]
- A **model** is the compiled, usable class derived from a schema that maps to a MongoDB collection and provides the APIs to create, read, update, and delete documents.[^1]
- A **query** is the chainable interface for building and executing database queries, mirroring MongoDB operations while adding mongoose convenience methods.[^1]

***

### Defining the user schema and model

- Create a `user.js` file, require mongoose, and define a schema via `new mongoose.Schema({ ... })`, saving it in a variable like `userSchema` for reuse.[^1]
- Keys in the schema object correspond to document fields; for example, `name: String` and `age: Number` define simple fields with built‑in types, with many more types covered later in the lecture.[^1]
- Compile the model with `mongoose.model('User', userSchema)`, where `'User'` determines the collection name (pluralized by mongoose), and export the model with `module.exports = ...` for use in other files.[^1]

```js
// user.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: String,
  age: Number
});

module.exports = mongoose.model('User', userSchema);
```


***

### Creating and saving documents

- In `script.js`, require the user model with `const User = require('./user')` and instantiate with `new User({ name: 'kyle', age: 26 })` to create a local instance not yet persisted.[^1]
- Persist changes using `await user.save()` (inside an `async` function) or `.then(...)` on the returned promise; the recommended approach is wrapping logic in an `async function run() { ... }` and calling `run()`.[^1]
- Logging the saved user shows an auto‑generated `_id` and a `__v` version key managed by mongoose internally, indicating a successful persist of the object.[^1]

```js
async function run() {
  const user = new User({ name: 'kyle', age: 26 });
  await user.save();
  console.log(user);
}
run();
```


***

### Alternative creation via Model.create

- `User.create({...})` is an alternative to `new User(...).save()`, returning the created document and performing the same persist under the hood.[^1]
- Both approaches support `await` for ergonomic async control flow and provide identical saved results including `_id` and `__v` fields.[^1]

```js
const user = await User.create({ name: 'kyle', age: 26 });
```


***

### Updating a document instance

- Updating the in‑memory instance (e.g., `user.name = 'sally'`) does not persist automatically; calling `await user.save()` writes the change to the database.[^1]
- This workflow—fetch, mutate fields on the document, then `save()`—is the recommended way to ensure all validations and middleware run consistently.[^1]

```js
user.name = 'sally';
await user.save();
```


***

### Expanding schema: types, arrays, subdocuments, and references

- Add fields such as `email: String`, `createdAt: Date`, `updatedAt: Date`, `hobbies: [String]` (array of strings), and a nested `address: { street: String, city: String }` to demonstrate more complex shapes.[^1]
- Represent references via `bestFriend: mongoose.Schema.Types.ObjectId` to point at another user’s `_id`, enabling relations where a user references another user.[^1]
- Subdocuments can be defined inline as nested objects or as separate schemas; defining a separate `addressSchema` and using it inside `userSchema` generates an `_id` for the subdocument and is useful for complex nested structures.[^1]

```js
const addressSchema = new mongoose.Schema({
  street: String,
  city: String
});

const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
  email: String,
  createdAt: Date,
  updatedAt: Date,
  bestFriend: mongoose.Schema.Types.ObjectId,
  hobbies: [String],
  address: addressSchema
});
```


***

### Type casting errors and error handling

- Mongoose validates types at save/create time, so assigning a string to a numeric field like `age: "26"` throws a cast error that can be caught and inspected.[^1]
- Wrap logic in `try/catch` and log `e.message` for a user‑friendly string, or inspect rich details on `e.errors.<field>` for low‑level diagnostics like path, kind, and value.[^1]
- The lecture notes that plugins exist to improve error messages presentation, though the built‑in messages already carry actionable details for development.[^1]

```js
try {
  await User.create({ name: 'kyle', age: "26" });
} catch (e) {
  console.log(e.message);
  // console.log(e.errors.age) // detailed error info
}
```


***

### Required, transforms, and defaults

- Use `{ type: String, required: true }` for required fields like `email`, which rejects saves when the field is missing and surfaces a “required” validation error.[^1]
- Use `{ lowercase: true }` to normalize `email` to lowercase automatically, or `{ uppercase: true }` if uppercase normalization is desired instead.[^1]
- Supply defaults with `{ default: Date.now }` on `createdAt` and `updatedAt`; passing a function (like `Date.now`) ensures a fresh value each time a new document is created or saved, whereas `new Date()` at schema build time would freeze the value.[^1]

```js
email: { type: String, required: true, lowercase: true },
createdAt: { type: Date, default: Date.now },
updatedAt: { type: Date, default: Date.now }
```


***

### Immutable fields

- Mark fields like `createdAt` as `{ immutable: true }` so attempts to mutate and save the document do not alter that value, with no thrown error but with the mutation silently ignored.[^1]
- This is ideal for timestamps or identifiers that must never change after creation, preserving data integrity regardless of application‑side mutations.[^1]

```js
createdAt: { type: Date, default: Date.now, immutable: true }
```


***

### Numeric and string bounds

- Enforce numeric bounds with `min` and `max` on numbers, such as `age: { type: Number, min: 1, max: 100 }`, producing validation errors when values are outside the allowed range.[^1]
- Enforce string length bounds with `minLength` and `maxLength` on strings, such as `email: { type: String, minLength: 10 }`, producing length‑specific validation errors when violated.[^1]

```js
age: { type: Number, min: 1, max: 100 },
email: { type: String, minLength: 10 }
```


***

### Custom validation

- Define custom validators using `{ validate: { validator(value) { ... }, message(props) { ... } } }` to encapsulate domain logic like “age must be even,” returning true/false to allow or reject the value.[^1]
- Example: `age % 2 === 0` with message `"{props.value} is not an even number"` rejects odd ages and surfaces a specific, readable error.[^1]
- Centralizing custom validations at the model layer avoids duplicating validation checks throughout the codebase and ensures consistent enforcement.[^1]

```js
age: {
  type: Number,
  validate: {
    validator: (v) => v % 2 === 0,
    message: (props) => `${props.value} is not an even number`
  }
}
```


***

### Important validation caveat

- Mongoose validations (built‑in and custom) run when using `create()` or `save()`, but do not run for direct database operations like `updateOne`, `updateMany`, `findOneAndUpdate`, or `findByIdAndUpdate`.[^1]
- The recommended pattern is: find the document (`findById` or `findOne`), mutate fields on the instance, and then call `save()` so all validations and middleware are enforced.[^1]
- Avoid the “find and update” shortcuts that bypass validation unless intentionally opting out of model‑level validation guarantees for a specific, controlled reason.[^1]

***

### Reading documents

- `findById(id)` retrieves a single document by `_id`, returning a document instance or `null` if not found, and supports `await` for async handling.[^1]
- `find({ ... })` uses MongoDB‑like query syntax to return an array, while `findOne({ ... })` returns the first match, providing a smoother ORM experience atop MongoDB queries.[^1]
- `exists({ ... })` checks if at least one document matches the filter, returning a truthy value, useful for quick presence checks without fetching full documents.[^1]

```js
const one = await User.findById(id);
const many = await User.find({ name: 'kyle' });
const any = await User.exists({ name: 'kyle' });
```


***

### Deleting documents

- `deleteOne({ ... })` deletes the first matching document and returns a result object with `deletedCount`, while `deleteMany({ ... })` removes all matching documents.[^1]
- Logging the result object reveals whether a deletion occurred, which is helpful for asserting side effects in scripts and tests.[^1]

```js
const result = await User.deleteOne({ name: 'kyle' });
// result.deletedCount === 1
```


***

### Query builder (chainable “where” style)

- Mongoose queries can be built fluently: `User.where('name').equals('kyle').where('age').gt(12)` mirrors MongoDB operators with chainable helpers like `.gt()`, `.lt()`, `.limit()`, and `.select()`.[^1]
- Conditions can be chained arbitrarily, for example constraining an age range while matching a specific name, and adding a `.limit(2)` to return only two results.[^1]
- `.select('age')` limits returned fields, while other variants also support excluding fields or combining multiple selections per query.[^1]

```js
const q = await User
  .where('name').equals('kyle')
  .where('age').gt(12).lt(31)
  .limit(2)
  .select('age');
```


***

### References and populate (joins made easy)

- Set `bestFriend` to an object id of another user and then save, establishing a reference relationship at the document level.[^1]
- Define the reference target with `bestFriend: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }` in the schema to enable population.[^1]
- Use `.populate('bestFriend')` to replace the stored id with the full referenced document at query time, functioning similarly to a join in relational databases but handled client‑side by mongoose.[^1]

```js
// schema
bestFriend: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }

// usage
const user = await User.findOne({ ... }).populate('bestFriend');
```


***

### Instance methods

- Add methods available on every document instance by attaching to `userSchema.methods`, using regular functions (not arrow functions) so `this` refers to the document.[^1]
- Example: `sayHi` prints the instance’s name with `this.name`, allowing reusable per‑document behaviors without scattering logic throughout the codebase.[^1]
- After defining, fetched or newly created documents have `.sayHi()` available, e.g., `const user = await User.findOne(...); user.sayHi()`.[^1]

```js
userSchema.methods.sayHi = function () {
  console.log(`Hi my name is ${this.name}`);
};
```


***

### Static methods

- Add model‑level methods via `userSchema.statics` for operations that conceptually belong to the model rather than a single instance, such as a custom “find by name” helper.[^1]
- A typical implementation normalizes input and returns a query: for example, `findByName(name)` that performs a case‑insensitive match with a regex.[^1]
- Static methods are invoked on the model: `const users = await User.findByName('kyle')`, returning a query or results depending on composition.[^1]

```js
userSchema.statics.findByName = function (name) {
  return this.where({ name: new RegExp(name, 'i') });
};
```


***

### Query helpers

- Attach methods directly to queries with `userSchema.query`, which makes the helper available on the chain returned by `find`/`where` calls rather than on the model itself.[^1]
- Example: `.byName(name)` that applies a case‑insensitive regex filter and can be composed with other query helpers and constraints in a single chain.[^1]
- Use as `await User.find().byName('kyle')`, noting that query helpers cannot be called on the model directly (i.e., `User.byName` is invalid).[^1]

```js
userSchema.query.byName = function (name) {
  return this.where({ name: new RegExp(name, 'i') });
};
```


***

### Virtuals

- Define derived properties not stored in the database with `userSchema.virtual('namedEmail')` and a `.get(function () { ... })`, ensuring a regular function so `this` refers to the document.[^1]
- Example virtual: combine `name` and `email` into `"Name <email>"`, which reads from current document fields without duplicating data in storage.[^1]
- Virtuals appear on document instances in code—`user.namedEmail`—but do not persist to MongoDB, allowing safe reuse across the application.[^1]

```js
userSchema.virtual('namedEmail').get(function () {
  return `${this.name} <${this.email}>`;
});
```


***

### Middleware (pre/post hooks)

- Insert logic before or after operations using middleware like `pre('save')`, `post('save')`, `pre('validate')`, and `pre('remove')` to centralize side effects and constraints.[^1]
- A common pattern is updating timestamps: in `pre('save')` set `this.updatedAt = Date.now()` and call `next()` to proceed, ensuring modifications are always current on save.[^1]
- In `post('save')`, the handler receives the saved document as the first parameter; calling `doc.sayHi()` demonstrates chaining document methods after persistence.[^1]

```js
userSchema.pre('save', function (next) {
  this.updatedAt = Date.now();
  next();
});

userSchema.post('save', function (doc, next) {
  doc.sayHi();
  next();
});
```


***

### Aborting in pre hooks

- Throwing an error inside `pre('save')` (e.g., `throw new Error('fail save')`) aborts the operation, which can be caught in application code with `try/catch` to surface the failure.[^1]
- Omitting `next()` in a `pre` hook halts the chain, but explicitly throwing an error is clearer and integrates with error handling pathways.[^1]
- This mechanism applies to other supported middleware points, enabling domain rules to guard persistence and ensure consistent data quality.[^1]

```js
userSchema.pre('save', function () {
  throw new Error('fail save');
});
```


***

### Putting it together: example snippets

- Adding hobbies and nested address to the user instance allows demonstration of arrays and subdocuments, such as `hobbies: ['weightlifting', 'bowling']` and `address: { street: 'Main Street' }`.[^1]
- Saving after property changes (`await user.save()`) shows persisted nested structures and arrays in the console output, validating schema structure and defaults.[^1]
- Switching `address` from an inline object to a dedicated `addressSchema` demonstrates the generation of an `_id` for the subdocument, useful for more complex nested behaviors.[^1]

```js
const user = await User.create({
  name: 'kyle',
  age: 26,
  hobbies: ['weightlifting', 'bowling'],
  address: { street: 'Main Street', city: 'Example' }
});
```


***

### Caution on update shortcuts

- Methods like `findOneAndUpdate`, `findByIdAndUpdate`, `updateOne`, and `updateMany` operate directly on the database and bypass mongoose validation and certain middleware, unlike `save()`.[^1]
- To ensure validations and hooks run, prefer the pattern: find the document, modify fields on it, and call `save()`, which centralizes business logic at the schema/model layer.[^1]
- Delete helpers (`deleteOne`, `deleteMany`) are often acceptable since they do not mutate documents, but still be conscious of any remove hooks or cascading rules if defined.[^1]

***

### Final notes and resources

- The lecture closes by reiterating that mastering mongoose validations, query builders, population, methods, virtuals, and middleware elevates application code quality and maintainability beyond basic CRUD usage.[^1]
- It mentions a free MongoDB cheat sheet download as a helpful companion for both MongoDB and mongoose workflows, encouraging use alongside these patterns and techniques.[^1]
- With these tools, code benefits from centralized schema rules, clear error handling, and composable queries that mirror MongoDB while providing higher‑level developer ergonomics.[^1]

***

## Appendix: Common code patterns from the lecture

```js
// Connect
mongoose.connect('mongodb://localhost/testdb',
  () => console.log('connected'),
  err => console.log(err)
);

// User schema with refs, arrays, subdoc, validators, defaults
const addressSchema = new mongoose.Schema({ street: String, city: String });

const userSchema = new mongoose.Schema({
  name: String,
  age: {
    type: Number,
    min: 1,
    max: 100,
    validate: {
      validator: v => v % 2 === 0,
      message: props => `${props.value} is not an even number`
    }
  },
  email: { type: String, required: true, lowercase: true, minLength: 10 },
  createdAt: { type: Date, default: Date.now, immutable: true },
  updatedAt: { type: Date, default: Date.now },
  bestFriend: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  hobbies: [String],
  address: addressSchema
});

// Methods, statics, query helpers, virtuals
userSchema.methods.sayHi = function () {
  console.log(`Hi my name is ${this.name}`);
};
userSchema.statics.findByName = function (name) {
  return this.where({ name: new RegExp(name, 'i') });
};
userSchema.query.byName = function (name) {
  return this.where({ name: new RegExp(name, 'i') });
};
userSchema.virtual('namedEmail').get(function () {
  return `${this.name} <${this.email}>`;
});

// Middleware
userSchema.pre('save', function (next) {
  this.updatedAt = Date.now();
  next();
});
userSchema.post('save', function (doc, next) {
  doc.sayHi();
  next();
});

const User = mongoose.model('User', userSchema);

// Usage
(async function run() {
  try {
    const user = await User.create({ name: 'kyle', age: 26, email: 'Test@Test.com' });
    user.name = 'sally';
    await user.save();

    const found = await User.find().byName('kyle').limit(2).select('age');
    const one = await User.findById(user._id).populate('bestFriend');

    console.log(user.namedEmail);
  } catch (e) {
    console.log(e.message);
  }
})();
```


***

If a version with callouts, color‑coded badges, or an index is preferred, this can be layered on top without changing the preserved content or examples in this documentation.[^1]

<div align="center">⁂</div>

[^1]: paste.txt

