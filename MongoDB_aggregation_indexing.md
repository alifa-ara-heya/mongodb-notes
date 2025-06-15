# MongoDB Aggregation Notes

A helpful collection of MongoDB query examples with syntax, explanations, and usage. This is for my future reference.

---

## 📋 Table of Contents

## 1. `$match`, `$project` aggregation stage

```js
db.test.find()
// same as
db.test.aggregate([])

//stage-1
db.test.find(
    {
        gender: "Male"
    })

//or with aggregation
db.test.aggregate([
    { $match: { gender: 'Male' } }
])

// conditional filtering
db.test.find({
    gender: "Male", age: { $lte: 30 }
})

// with aggregation (same)
db.test.aggregate([
    { $match: { gender: 'Male', age: { $lte: 30 } } }
])


// projection
db.test.aggregate([
    //stage-1
    { $match: { gender: 'Male', age: { $lte: 30 } } },
    // stage-2
    {$project: {name: 1, age: 1, gender: 1}}
])

// or
db.test.aggregate([
    //stage-1
    { $match: { gender: 'Male' } },
    // stage-2
    { $match: { age: { $lte: 30 } } },
    // stage-3
    { $project: { name: 1, age: 1, gender: 1 } }
])

```

## 2. `$addFields`, `$out`, `$merge` aggregation stage

```js
//this does not modify the original document. Only adds fields in the context of the pipeline
db.test.aggregate([
    //stage-1
    { $match: { gender: 'Male' } },
    // stage-2
    { $addFields: { course: "level-2", eduTech: "Programming hero" } },
    // stage-3
    { $project: { course: 1, eduTech: 1 } },
    // stage-4- $out makes a new collection 
    {$out: "course-students"}
])


// but if we want to merge the new fields with our previous document. we use $merge operator
db.test.aggregate([
    //stage-1
    { $match: { gender: 'Male' } },
    // stage-2
    { $addFields: { course: "level-2", eduTech: "Programming hero" } },
    // stage-3
    // { $project: { course: 1, eduTech: 1 } },
    // stage-4
    { $merge: "test" }
])







```
