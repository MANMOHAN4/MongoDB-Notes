# MongoDB Shell Basics: Full Notes

---

## **Overview**

This document reformats the professor’s lecture transcript into clear, readable notes you can study like documentation. It preserves all commands, examples, terminology, and explanations, without condensing content. The shell referenced is the **sh terminal** (MongoDB shell-like environment). The examples use a database called **appdb** and a **users** collection.

---

## **Starting in the Default Database**

- When you open the sh terminal, you may see the word `test` in the prompt.
- `test` is the default database you start in. It has no data and does not really exist until you create data.

---

## **View Databases**

- To view your databases:

```javascript
show dbs
```

- You will typically see **admin**, **config**, and **local**.
- If you already created a database (for example `appdb`) and it has data, it will also appear in this list. Otherwise, you won’t see it.

---

## **Use or Create a Database**

- To access and use a database:

```javascript
use appdb
```

- You can substitute `appdb` with any new name to create a new database and switch into it.
- After using a database, your prompt will display that database name (e.g., `appdb` instead of `test`).

---

## **View Collections**

- To view what collections exist in the current database:

```javascript
show collections
```

- In MongoDB:
  - **Databases** are like databases in SQL.
  - **Collections** are like tables inside a database.
  - **Documents** are like rows; they are JSON-like objects stored in collections.

---

## **Drop a Database**

- To delete the entire current database:

```javascript
db.dropDatabase();
```

- You’ll see a result indicating it dropped the database, for example `ok 1` and which database was dropped.
- Verifying removal:

```javascript
show dbs
```

- The dropped database will no longer appear.

---

## **Clear the Screen**

- To clear the terminal screen:

```javascript
cls;
```

- This clears existing output for a fresh view.

---

## **Auto-Creation Behavior**

- MongoDB is interesting compared to SQL in that you **don’t need to explicitly create databases, users, or collections**.
- If you try to access something that doesn’t exist and you insert data into it, MongoDB will **automatically create** the database and collection.
- For example, even if `show dbs` does not list `appdb`, inserting into it will create it.

---

## **Exit and Re-enter the Terminal**

- To exit the terminal:

```javascript
exit;
```

- To re-enter, start the sh terminal again.

---

## **Confirming and Switching to appdb**

- In the terminal:

```javascript
use appdb
show dbs
```

- The database may not exist yet; inserting data will create it.
- To access the current database object:

```javascript
db;
```

---

## **Insert One Document**

- Access a collection by referencing it as a property on `db`. For example:

```javascript
db.users.insertOne({ name: "john" });
```

- Output indicates the insert was acknowledged and shows the generated **\_id**.
- MongoDB automatically generates a unique **\_id** for each document.

---

## **Find Documents**

- To get all documents in a collection:

```javascript
db.users.find();
```

- You’ll see an array of documents with **\_id** and any fields you inserted (e.g., `name: "john"`).

---

## **Documents, Collections, and No Schema**

- MongoDB stores data as **documents** inside **collections** inside **databases**.
- There are **no fixed schemas or required columns**. You can insert JSON-like documents with varying fields.
- Example: you can insert one document with only a name, and another with name, age, nested address, and an array of hobbies.

---

## **Insert a Nested Document and an Array**

- Example inserting sally with more fields and nested data:

```javascript
db.users.insertOne({
  name: "sally",
  age: 19,
  address: { street: "987 North Street" },
  hobbies: ["running"],
});
```

- Verify:

```javascript
db.users.find();
```

- You will see **john** and **sally**; sally has name, age, address with street, and an array of hobbies.

---

## **Insert Many**

- Insert multiple records at once with an array of objects:

```javascript
db.users.insertMany([{ name: "jill" }, { name: "mike" }]);
```

- Verify:

```javascript
db.users.find();
```

- You now also see **jill** and **mike**.

---

## **Add More Sample Data for Queries**

- Insert more complex users for querying:

```javascript
db.users.insertMany([
  {
    name: "kyle",
    age: 26,
    hobbies: ["weightlifting", "bowling"],
    address: { street: "123 Main Street" },
  },
  {
    name: "billy",
    age: 41,
    hobbies: ["swimming", "bowling"],
    address: { street: "456 Oak Avenue" },
  },
]);
```

- If you make a typo or syntax error, the shell provides helpful error messages indicating what was wrong.
- Verify:

```javascript
db.users.find();
```

- You should now see **kyle** (age 26, hobbies weightlifting and bowling, address) and **billy** (age 41, hobbies swimming and bowling, address), among the other users.

---

## **Sorting, Limiting, and Skipping**

- Get all:

```javascript
db.users.find();
```

- **Limit** results:

```javascript
db.users.find().limit(2);
```

- **Sort** by a field:

```javascript
db.users.find().sort({ name: 1 }).limit(2); // ascending by name
db.users.find().sort({ name: -1 }).limit(2); // descending by name
```

- **Sort by multiple fields:**

```javascript
db.users.find().sort({ age: 1, name: -1 });
db.users.find().sort({ age: -1, name: -1 });
```

- **Skip** results:

```javascript
db.users.find().limit(2); // first two
db.users.find().skip(1).limit(2); // skip the first; get the next two
```

---

## **Basic Filtering (Where-like Queries)**

- **Equality by field:**

```javascript
db.users.find({ name: "kyle" });
db.users.find({ age: 26 });
```

- **Types matter.** If age is a number, a string “26” will not match:

```javascript
db.users.find({ age: "26" }); // returns nothing if age is stored as a number
```

---

## **Projections (Select Specific Fields)**

- **Include only name and age for a single match:**

```javascript
db.users.find({ name: "kyle" }, { name: 1, age: 1 });
```

- **Exclude \_id:**

```javascript
db.users.find({ name: "kyle" }, { name: 1, age: 1, _id: 0 });
```

- **Exclude a field (return all other fields):**

```javascript
db.users.find({ name: "kyle" }, { age: 0 });
```

---

## **Complex Operators: \$eq and \$ne**

- **Explicit equality with \$eq:**

```javascript
db.users.find({ name: { $eq: "sally" } });
```

- **Not equal with \$ne:**

```javascript
db.users.find({ name: { $ne: "sally" } });
```

---

## **Range Comparisons on Numbers**

- **Greater than / greater than or equal / less than / less than or equal:**

```javascript
db.users.find({ age: { $gt: 13 } });
db.users.find({ age: { $gte: 19 } });
db.users.find({ age: { $lte: 19 } });
db.users.find({ age: { $lt: 19 } });
```

---

## **\$in and \$nin (Set Membership)**

- **In a list of values:**

```javascript
db.users.find({ name: { $in: ["kyle", "sally"] } });
```

- **Not in a list:**

```javascript
db.users.find({ name: { $nin: ["kyle", "sally"] } });
```

---

## **\$exists to Check Presence of a Key**

- **Documents where age exists:**

```javascript
db.users.find({ age: { $exists: true } });
```

- **Documents where age does not exist:**

```javascript
db.users.find({ age: { $exists: false } });
```

- **Note:** \$exists only checks whether the key exists, not the value. A document with `age: null` still “exists” for age.
- **Demonstration:**

```javascript
db.users.insertOne({ name: "bill", age: null });
db.users.find({ age: { $exists: true } }); // includes bill with age: null
```

---

## **Combine Comparisons (Implicit AND)**

- **Range on a single field (implicit AND inside the object):**

```javascript
db.users.find({
  age: { $gte: 20, $lte: 40 },
});
```

- **Combine with another field (AND across fields):**

```javascript
db.users.find({
  age: { $gte: 20, $lte: 40 },
  name: "kyle",
});
```

---

## **Explicit \$and**

- You rarely need \$and because AND is implicit, but it exists:

```javascript
db.users.find({
  $and: [{ age: 26 }, { name: "kyle" }],
});
```

---

## **\$or (No Alternative Without It)**

- **One of multiple conditions must be true:**

```javascript
db.users.find({
  $or: [{ age: { $lte: 20 } }, { name: "kyle" }],
});
```

- This returns **sally** (age <= 20) and **kyle** (name is kyle).

---

## **\$not (Negation of a Condition)**

- **Negate a condition by wrapping it:**

```javascript
db.users.find({
  age: { $not: { $lte: 20 } },
});
```

- This returns users where age is **not less than or equal to 20**, and also returns users where age is missing or null.
- **Compare to a direct greater-than:**

```javascript
db.users.find({ age: { $gt: 20 } });
// Only users with a defined age > 20 are returned (missing ages are excluded)
```

- Generally you don’t need \$not because \$ne, \$gt, \$lt, etc. already cover most “not” cases, but it’s good to know.

---

## **Compare Two Fields with \$expr**

- **Insert users with balance and debt:**

```javascript
db.users.insertMany([
  { name: "userA", balance: 100, debt: 200 },
  { name: "userB", balance: 500, debt: 100 },
]);
```

- **Find users where debt is greater than balance:**

```javascript
db.users.find({
  $expr: { $gt: ["$debt", "$balance"] },
});
```

- **Important:** When comparing fields, prefix field names with `$` (e.g., `$debt` and `$balance`).

---

## **Query Nested Fields**

- **Use dot notation in quotes for nested fields (just like accessing in JavaScript):**

```javascript
db.users.find({ "address.street": "Main Street" });
```

- You can also use comparison operators on nested fields.

---

## **findOne and countDocuments**

- **Return just the first matching document:**

```javascript
db.users.findOne({ age: { $lte: 40 } });
```

- **Count how many documents match a query:**

```javascript
db.users.countDocuments({ age: { $lte: 40 } });
```

- For example, this might return 2 if there are two users with age <= 40.

---

## **Updating Data: updateOne**

- **Update the first matching document:**

```javascript
db.users.updateOne(
  { age: 26 }, // filter (same syntax as find)
  { $set: { age: 27 } } // update uses $ operators (required)
);
```

- Using a field assignment without an atomic operator throws an error:
  - “update document requires atomic operators.” Use `$set` for setting fields.
- **Verify:**

```javascript
db.users.findOne({ age: 27 });
```

- **Update by \_id (common pattern):**

```javascript
db.users.updateOne(
  { _id: ObjectId("...paste-id-here...") },
  { $set: { name: "new name" } }
);
db.users.findOne({ _id: ObjectId("...paste-id-here...") });
```

---

## **Other Useful Update Operators**

- **Increment a numeric field:**

```javascript
db.users.updateOne(
  { _id: ObjectId("...") },
  { $inc: { age: 3 } } // e.g., 27 -> 30
);
```

- **Rename a field:**

```javascript
db.users.updateOne(
  { _id: ObjectId("...") },
  { $rename: { name: "firstName" } }
);
```

- **Unset (remove) a field:**

```javascript
db.users.updateOne(
  { _id: ObjectId("...") },
  { $unset: { age: "" } } // value can be empty string; field is removed, not set to null
);
```

- **Push into an array:**

```javascript
db.users.updateOne(
  { _id: ObjectId("...") },
  { $push: { hobbies: "swimming" } }
);
```

- **Pull (remove) values from an array:**

```javascript
db.users.updateOne({ _id: ObjectId("...") }, { $pull: { hobbies: "bowling" } });
```

- **push and pull accept query-like expressions.** For example, with numeric arrays, you could remove everything less than or equal to 3.

---

## **Update Many Documents**

- **Apply an update to all documents matching a filter:**

```javascript
db.users.updateMany(
  { address: { $exists: true } },
  { $unset: { address: "" } }
);
```

- The result indicates how many documents were modified.
- **Verify:**

```javascript
db.users.find();
```

- You will see none of the users have an address after this operation (if the result said it updated three, you’ll see three removed).

---

## **replaceOne**

- **Replace a document entirely:**

```javascript
db.users.replaceOne(
  { age: 30 }, // finds the document (e.g., kyle)
  { name: "john" } // replaces the entire object with this new one
);
```

- **Verify:**

```javascript
db.users.find({ name: "john" });
```

- `replaceOne` deletes everything at that document and replaces it with what you provide. Usually, you want `update` (99.9% of the time) instead of replace, so you only change specific fields.

---

## **Deleting Documents**

- **Delete one:**

```javascript
db.users.deleteOne({ name: "john" });
db.users.find({ name: "john" }); // returns nothing
```

- **Delete many:**

```javascript
db.users.deleteMany({ age: { $exists: false } });
```

- This deletes all documents that do not have an age field. If a document has `age: null`, it still has the key, so it will not be deleted by `$exists: false`.
- After deletion, you’ll only see users that have an age key in their document.

---

## **Notes on Shell Behavior and Errors**

- If you mistype a command or punctuation, the shell shows clear error messages that help you correct the issue.
- Many methods and operators across find, update, arrays, etc., use consistent patterns: filters are objects; complex filters use `$` operators; updates use atomic operators like `$set`, `$inc`, `$rename`, `$unset`, `$push`, and `$pull`.

---

## Summary of MongoDB Data Model

- **Database**: Top-level container (e.g., `appdb`).
- **Collection**: Like a table in SQL (e.g., `users`).
- **Document**: JSON-like object stored in a collection (e.g., `{ name: "John" }`).
- **No schema**: Documents can have different fields and nested structures.

---

## **Summary of Key Ideas Used**

- `show dbs`, `use <db>`, `show collections`, `db.dropDatabase()`, `cls`, `exit`
- `db` and collection access via `db.<collection>`
- `insertOne`, `insertMany` for creating documents
- `find`, `findOne`, `sort`, `limit`, `skip`, projections (include/exclude), `countDocuments`
- Equality and complex operators: `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin`, `$exists`, `$and`, `$or`, `$not`, `$expr`
- Nested field queries with dot notation
- Update operations: `updateOne`, `updateMany` with `$set`, `$inc`, `$rename`, `$unset`, `$push`, `$pull`
- `replaceOne` for full replacement (use sparingly)
- `deleteOne`, `deleteMany` for deletions

---

## Key MongoDB Shell Commands Reference

| Command                        | Description                          |
| :----------------------------- | :----------------------------------- |
| `show dbs`                     | List all databases                   |
| `use <db>`                     | Switch to (or create) a database     |
| `show collections`             | List collections in current database |
| `db.dropDatabase()`            | Delete current database              |
| `cls`                          | Clear the shell screen               |
| `exit`                         | Exit the shell                       |
| `db`                           | Show current database                |
| `db.<collection>.find()`       | Find all documents in a collection   |
| `db.<collection>.insertOne()`  | Insert one document                  |
| `db.<collection>.insertMany()` | Insert multiple documents            |
| `db.<collection>.updateOne()`  | Update one document                  |
| `db.<collection>.updateMany()` | Update multiple documents            |
| `db.<collection>.replaceOne()` | Replace one document                 |
| `db.<collection>.deleteOne()`  | Delete one document                  |
| `db.<collection>.deleteMany()` | Delete multiple documents            |

---

## Final Notes

- MongoDB shell commands are similar to JavaScript function calls.
- You do not need to predefine schemas, tables, or columns.
- Documents are flexible and can be nested or contain arrays.
- Error messages are clear and help you correct mistakes quickly.
- Most queries and update operators start with a `$` (dollar sign).

This documentation covers all the basics and advanced queries, updates, and deletions you need to work with MongoDB in the shell.
<span style="display:none">[^1]</span>

<div align="center">⁂</div>

[^1]: i-want-to-talk-about-are-just-basic-commands-that.md
