title: 'MongoDB: Cannot change the size of a document in a capped collection'
tags:
  - MongoDB
date: 2016-03-26 14:34:15
---
### Error code 10003

We encountered an unexpected error while trying to update a [capped collection](https://docs.mongodb.org/manual/core/capped-collections/) in MongoDB after upgrading from 3.0 to 3.2:


~~~javascript

Mongo> db.logItems
   .update({_id : ObjectId('566839ad1d8bd6870a1ce5c2')},
           {'$set' : {'processed' : true}})
WriteResult({
        "nMatched" : 0,
        "nUpserted" : 0,
        "nModified" : 0,
        "writeError" : {
                "code" : 10003,
                "errmsg" : "Cannot change the size of a document
                 in a capped collection: 28320 != 28331"
        }
})
~~~


As it turns out this behaviour is to be expected. The [documentation](https://docs.mongodb.org/manual/core/capped-collections/#restrictions-and-recommendations) clearly states that the size of a document in a capped collection can not be changed.

<!-- more-->

So why didn't we encounter the same issue while using MongoDB 3.0? There was a bug in 3.0 which [allowed capped collection objects to grow](https://jira.mongodb.org/browse/SERVER-20529) with the WiredTiger storage engine. Version 3.2 fixes the error, which caused our code to fail.

### What we dit to fix the error
When saving a new document add the 'processed' field and initialize it to false. Adding a property changes the size of the document, but changing a boolean from false to true doesn't affect the size.

Create a script to migrate the existing documents, adding the processed field as required:

~~~javascript
db.createCollection( "logItems_temp", { capped: true, size: 100000 } )
var cur = db.logItems.find()
while (cur.hasNext()) {
  logItem = cur.next(); 
  if(logItem.processed == null){
	  logItem.processed = false;
  }
  db.logItems_temp.insert(logItem);
}
db.logItems.drop()
db.logItems_temp.renameCollection("logItems")
~~~
