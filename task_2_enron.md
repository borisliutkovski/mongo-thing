#### Which pair of people have the greatest number of messages in the dataset?

```
db.enron.aggregate([
    { $unwind: "$headers.To" },
    { $group: { _id: { id: "$_id", from: "$headers.From" }, to: { $addToSet:"$headers.To" } } },
    { $unwind: "$to" },
    { $group: { _id: { from: "$_id.from", to: "$to" }, count: { $sum: 1 } } },
    { $sort: { count: -1 } },
    { $limit: 1 },
    { $project: { _id: 0, from: "$_id.from", to: "$_id.to", count: "$count" } }
])
```
```
{ "from" : "susan.mara@enron.com", "to" : "jeff.dasovich@enron.com", "count" : 750 }
```
