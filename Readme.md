
# MongoDB Query Notes

A helpful collection of MongoDB query examples with syntax, explanations, and usage. Useful for beginners and intermediate developers.

---

## 📋 Table of Contents

1. [Basic Commands](#1-basic-commands)
2. [Insert Operations](#2-insert-operations)
3. [Find Operations](#3-find-operations)
4. [Projection](#4-projection)
5. [Operators](#5-operators)
    - [Comparison Operators](#comparison-opertaors)
    - [IN and NOT IN Operators](#in-and-not-in-operators)
6. [Sorting and Filtering](#sorting-and-filtering)

---

## 1. Basic Commands

```js
// Show all databases
show dbs

// Switch to 'testdb' database
use testdb

// Create a new collection
db.createCollection("test1")
```

## 2. Insert operations

```js
// Insert a single document
db.test.insertOne({ name: "sakib" })

// Insert multiple documents
db.test.insertMany([
  { name: "sakib" },
  { name: "Ayesha" }
])
```

## 3. Find operations

```js
// Find all documents
db.test.find()

// Find one document with specific field
db.test.findOne({ age: 21 })

// Using $eq operator
db.test.findOne({ gender: { $eq: "Male" } })
```

## 4. Projection

```js
// Project specific fields with find
db.test.find({ gender: "Male" }, { name: 1, gender: 1 })

// Chaining projection with .project()
db.test.find({ gender: "Male" }).project({ name: 1, gender: 1, email: 1 })

// Note: .project() doesn't work with findOne()
```

## 5. Operators

### Comparison opertaors

```js
// $eq example
db.test.find({ gender: { $eq: "Male" } })

// $gt and $lt together ("implicit and" with comma)
db.test.find({ age: { $gt: 18, $lt: 30 } }, { age: 1 })

// $gte (greater than or equal to)
db.test.find({ age: { $gte: 12 } }).sort({ age: 1 })

// $gte and $lte
db.test.find({ age: { $gte: 18, $lte: 30 } }, { age: 1 }).sort({ age: 1 })

// Combined gender + age filter
db.test.find({ gender: "Male", age: { $gte: 18, $lte: 30 } }, { age: 1, gender: 1 }).sort({ age: 1 })
```

---

### IN and NOT IN Operators

```js
// Using $in operator
db.test.find({
  gender: "Male",
  age: { $in: [18, 20, 24, 20] }
}, {
  age: 1, gender: 1
}).sort({ age: 1 })

// Using $nin operator (not in)
db.test.find({
  gender: "Male",
  age: { $nin: [18, 20, 24, 20] }
}, {
  age: 1, gender: 1
}).sort({ age: 1 })

// Using $nin and additional filter
db.test.find({
  gender: "Male",
  age: { $nin: [18, 20, 24, 20] },
  interests: "Cooking"
}, {
  age: 1, gender: 1, interests: 1
}).sort({ age: 1 })

// Using $nin and $in together on different fields
db.test.find({
  gender: "Male",
  age: { $nin: [18, 20, 24, 20] },
  interests: { $in: ["Cooking", "Gaming"] }
}, {
  age: 1, gender: 1, interests: 1
}).sort({ age: 1 })
```

---

### Logical Operators

```js
// $gt and $lt together ("implicit and" with comma)
db.test.find({ age: { $gt: 18, $lt: 30 } }, { age: 1 })

// explicit 'and'
db.test.find({
    $and: [
        { gender: "Female" },
        { age: { $ne: 15 } },
        { age: { $lte: 30 } },
        { age: { $gte: 18 } }
    ]
}).project({
    age: 1, gender: 1
}).sort({ age: 1 })


//Explicit $or
db.test.find({
    $or: [
        { interests: "Traveling" },
        {"skills.name": "JAVASCRIPT"}
    ]
}).project({
    interests: 1,
    "skills.name":1
}).sort({ age: 1 })


// Implicit $or with $in operator
db.test.find({ "skills.name": { $in: ["JAVASCRIPT", "PYTHON"] } }).project({
    "skills.name": 1
}).sort({ age: 1 })

```

---

### Element operators

```js
// $exists- if a field exixts in a document
db.test.find({ age: { $exists: false } })


// $type
db.test.find({ age: { $type: "string"} })

```

## 6. Sorting and Filtering

```js
// Sort by createdAt descending to get most recent
db.test.find({ gender: "Female" }).sort({ createdAt: -1 }).limit(1)

// Ascending sort
db.test.find({ age: { $gt: 12 } }).sort({ age: 1 })
```
