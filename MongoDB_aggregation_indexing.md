# MongoDB Aggregation Notes

A helpful collection of MongoDB query examples with syntax, explanations, and usage. This is for my future reference.

---

## 📋 Table of Contents

1. [`$match`, `$project` aggregation stage](#1-match-project-aggregation-stage)
2. [`$addFields`, `$out`, `$merge` aggregation stage](#2-addfields-out-merge-aggregation-stage)
3. [`$group`, `$sum`, `$push` aggregation stage](#3-group-sum-push-aggregation-stage)
4. [`$unwind` aggregation](#4-unwind-aggregation)
5. [`$bucket`, `$sort`, and `$limit` aggregation stage](#5-bucket-sort-and-limit-aggregation-stage)

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

## 3. `$group`, `$sum`, `$push` aggregation stage

```js
db.test.aggregate([
    //stage- 1
    { $group: { _id: "$address.country" } }
])


db.test.aggregate([
    //stage- 1
    { $group: { _id: "$age", count: { $sum: 1 } } } // count: { $sum: 1 } counts how many documents exist for each age. for example, there are 2 people whose age is 70
])

db.test.aggregate([
    //stage- 1
    { $group: { 
        _id: "$address.country", 
        count: { $sum: 1 }, 
        showName: {$push: "$name"} } }
])

// example output
/* 
{
 "_id" : "Norway",
 "count" : 1,
 "showName" : [
  {
   "firstName" : "Bobbye",
   "lastName" : "Smaile"
  }
 ]
}, */

// if we want to see everything based on full document
db.test.aggregate([
    //stage- 1
    {
        $group:
        {
            _id: "$address.country",
            count: { $sum: 1 },
            fullDoc: { $push: "$$ROOT" } //with the help of $$ROOT, one can use the full doc for the next stage
        }
    },
    //stage-2
    {
        $project: {
            "fullDoc.name": 1,
            "fullDoc.email": 1,
            "fullDoc.phone": 1,
        }
    }
])
// example output
/* {
 "_id" : "Norway",
 "fullDoc" : [
  {
   "name" : {
    "firstName" : "Bobbye",
    "lastName" : "Smaile"
   },
   "email" : "bsmailex@army.mil",
   "phone" : "(415) 1347163"
  }
 ]
}, */


// grouping by null makes the entire collection a document
db.test.aggregate([
    //stage- 1
    { 
        $group:  {
            _id: null,
            totalSalary: {$sum: 1} //will give the count of salary
        }
    }
])

// if we want to get the sum of salary and other operations
db.test.aggregate([
    //stage- 1
    {
        $group: {
            _id: null,
            totalSalary: { $sum: "$salary" },
            maxSalary: { $max: "$salary" },
            minSalary: { $min: "$salary" },
            avgSalary: { $avg: "$salary" },
        }
    },
    //stage-2
    {
        $project: {
            totalSalary: 1,
            maxSalary: 1,
            minSalary: 1,
            averageSalary: "$avgSalary",
            rangeBetweenMaxAndMin: { $subtract: ["$maxSalary", "$minSalary"] }
        }
    }
])

// output:
/* 
{
 "_id" : null,
 "totalSalary" : 30198,
 "maxSalary" : 499,
 "minSalary" : 105,
 "averageSalary" : 308.14285714285717,
 "rangeBetweenMaxAndMin" : 394
} */


```

## 4. `$unwind` aggregation

```js

// $unwind helps us to break the array and get individual elements
db.test.aggregate([
    //stage-1
    { $unwind: "$friends" },

    // stage-2
    {
        $group: { _id: "$friends", count: { $sum: 1 } }
    }
])

// if we want to see what are the interests in an age group-

db.test.aggregate([
    //stage-1
    { $unwind: "$interests" },

    // stage-2
    {
        $group: { _id: "$age", interestsPerAge: { $push: "$interests" } }
    }
])

```

## 5. `$bucket`, `$sort`, and `$limit` aggregation stage

```js
db.test.aggregate([
    //stage-1
    {
        $bucket: {
            groupBy: '$age',
            boundaries: [20, 40, 60, 80],
            default: "age group more than 80",
            output: {
                count: { $sum: 1 },
                people: { $push: "$$ROOT" }
            }
        }
    },
    //stage-2, sorting
    {
        $sort: { count: -1 }
    },
    //stage-3, limit (always use limit after sorting)
    {
        $limit: 2
    },
    //stage-4, projection
    {
        $project: { count: 1 }
    }
])

```

## 6. `$facet`, multiple pipeline aggregation stage

```js

db.test.aggregate([
    {
        $facet: {
            //pipeline-1
            "friendsCount": [
                //stage-1
                { $unwind: "$friends" },
                //stage-2
                { $group: { _id: "$friends", count: { $sum: 1 } } },
            ],
            // pipeline 2
            "educationCount": [
                //stage-1
                { $unwind: "$education" },
                //stage-2
                { $group: { _id: "$education", count: { $sum: 1 } } }
            ],
            //pipeline-3
            "skillsCount": [
                //stage-1
                { $unwind: "$skills" },
                //stage-2
                { $group: { _id: "$skills", count: { $sum: 1 } } }
            ]

        }
    }
])

```

## 7. `$lookup` - joining two collections- referencing

```js
// todo- difference between embedding and referencing

db.orders.aggregate([
    {
        $lookup: {
            from: "test",
            localField: "userId",
            foreignField: "_id",
            as: "customer"
        }
    }
])

// here, orders is a collection and customer is another collection

```

## 8. indexing, collscan vs ixscan

```js
db.test.find({_id : ObjectId("6406ad63fc13ae5a40000065")}).explain() //gives details about the process

db.test.find({_id : ObjectId("6406ad63fc13ae5a40000065")}).explain("executionStats")

//creating index over a single field
db.getCollection("massive-data").createIndex({email: 1})

//after indexing, the search time became much lower

```

## 9. compound index and text index

```js

// indexing over multiple fields

// dropping index
db.getCollection("massive-data").dropIndex({ email: 1 })

// creating text index
db.getCollection("massive-data").createIndex({ about: 'text' })

// searching text through this index
db.getCollection("massive-data").find({ $text: { $search: 'dolor' } })

```
