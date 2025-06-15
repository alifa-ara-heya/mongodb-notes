
# MongoDB Query Notes

A helpful collection of MongoDB query examples with syntax, explanations, and usage. This is for my future reference.

---

## 📋 Table of Contents

1. [Basic Commands](#1-basic-commands)
2. [Insert Operations](#2-insert-operations)
3. [Find Operations](#3-find-operations)
4. [Projection](#4-projection)
5. [Operators](#5-operators)
    - [Comparison Operators](#comparison-operators)
    - [`IN` and `NOT IN` Operators](#in-and-not-in-operators)
    - [Logical Operators](#logical-operators)
    - [Element operators](#element-operators)
    - [Array Query Operators](#array-query-operators)
6. [Sorting and Filtering](#6-sorting-and-filtering)
7. [Updating Documents](#7-updating-documents)
    - [Array update operators (`$addToSet`, `$push`)](#array-update-operators-addtoset-push)
8. [Deleting Document](#8-deleting-document)

---

## 1. Basic Commands

```js
// Show all databases
show dbs

// Switch to 'testDB' database
use testDB

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

### Comparison operators

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
// $exists- if a field exists in a document
db.test.find({ age: { $exists: false } })

// $type
db.test.find({ age: { $type: "string"} })

//to find the null values
db.test.find({company: {$type: "null"}}).project({company: 1})
db.test.find({company: null}).project({company: 1}) //either of these two is okay

// $size
db.test.find({friends: {$size: 4}}).project({friends: 1}) //gives you the friends array having 4 friends

db.test.find({friends: {$size: 0}}).project({friends: 1}) //shows friends with empty array

```

---

### Array Query Operators

```js
//finding an element in an array
db.test.find({ interests: "Cooking" }).project({ interests: 1 })

//finding an element in an array with index(position)
db.test.find({ "interests.2": "Cooking" }).project({ interests: 1 })

//finding an array of elements with exactly same position
db.test.find({ "interests": ["Gardening", "Reading", "Cooking"] }).project({ interests: 1 })

// finding an array of elements in any position
// $all operator
db.test.find(
    { "interests": { $all: ["Gardening", "Reading", "Cooking"] } })
    .project({ interests: 1 })

//finding an element in an array of object
db.test.find({ 
    "skills.name": "JAVASCRIPT"
}).project({skills: 1})

//gives documents containing exact match of field and values
db.test.find({
    "skills": {
        "name": "JAVASCRIPT",
        "level": "Expert",
        "isLearning": false
    }
}).project({ skills: 1 })

//if we want to match some elements of a field object
db.test.find({
    "skills": {
        $elemMatch: {
            "name": "JAVASCRIPT",
            "level": "Expert",
        }
    }
}).project({ skills: 1 })

```

## 6. Sorting and Filtering

```js
// Sort by createdAt descending to get most recent
db.test.find({ gender: "Female" }).sort({ createdAt: -1 }).limit(1)

// Ascending sort
db.test.find({ age: { $gt: 12 } }).sort({ age: 1 })
```

## 7. Updating Documents

```js
//finding a  document with its id
db.test.find({_id: ObjectId("6406ad63fc13ae5a40000065")})

// $set operator to modify/ update a document
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000065") },
    {
        $set: {
            age: 60
        }
    })


// $inc operator- to increase a value
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000066")},
    {
        $inc: {
                "age": 1
        }
    }
)

//updating an array with $set will replace the entire array, so for non-primitive data type such as arrays, we will not use $set

db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000065") },
    {
        $set: {
            interests: ["Gaming"] //changes the whole interests array
        }
    })
```

---

## Array update operators ($addToSet, $push)

```js
//$addToSet operator- to add an element in an already existing array
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000065") },
    {
        $addToSet: {
            interests: "Cooking"
        }
    })

// $each operator- adding multiple elements in an already existing array
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000065") },
    {
        $addToSet: {
            interests: {$each: ["Sewing", "Driving"]}
        }
    })

// But you cannot add duplicate values with $each and $addToSet operators
// $push operator- adds duplicate values 
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000065") },
    {
        $push: {
            interests: { $each: ["Sewing", "Driving"] }
        }
    })


// $unset- to remove the field from a document
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000065") },
    { $unset: { birthday: "" } } //removes the 'birthday' field
)

//or,
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000065") },
    { $unset: { age: 1 } } //removes the 'age' field
)

//$pop- removing the last element from an array
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000066") },
    { $pop: { friends: 1 } }
)

//$pop- removing the first element from an array
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000066") },
    { $pop: { friends: -1 } }
)

//$pull- removing a specific element from an array
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000066") },
    { $pull: { friends: 'Tanmoy Parvez' } }
)

//$pullAll- removing multiple elements from an array
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000066") },
    { $pullAll: { interests: ["Traveling", "Gaming"] } }
)

// more examples
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000065") },
    {
        $pullAll: {
            interests: [["Cooking", "Cookings"], {
                "each": ["Sewing", "Driving"]
            }]
        }
    }
)
```

## Updating Objects

```js
db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000065") },
    {
        $set: {
            "address.city": "dhaka",
             "address.country": "Bangladesh"
        }
    }
)

// updating array of objects

// The $ inside "education.$.major" is called the positional operator. It tells MongoDB to update the first matching element in the education array that satisfies the query condition.

db.test.updateOne(
    { _id: ObjectId("6406ad63fc13ae5a40000065"), "education.major": "Art" },
    {
        $set: {
                "education.$.major": "IT"
        }
    }
)

```

## 8. Deleting Document

```js
db.test.deleteOne({ _id: ObjectId("6406ad63fc13ae5a40000066") })

// creating a collection
db.createCollection("posts")

//inserting a document
db.posts.insertOne({ title: "My first post", content: "Hello MongoDB!" })

// deleting the entire collection
db.posts.drop()

```
