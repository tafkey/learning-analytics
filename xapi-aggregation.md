#Querying xAPU Activity Data

Whilst xAPI provides basic ways of querying data, this has not being sufficently flexible to provide access to data for most applications integrated as part of the Jisc project.  We therefore recommend using the Learning Locker Aggregration API.  

This is documented here: http://docs.learninglocker.net/statements_api/

## Sample Queries

The follow are examples of queries that can be created using the aggregation API. These are real example from the Jisc student app.

### Example one - Activity Type count

Count of different activity verbs between two dates:

[ { "$match":{ "statement.actor.name":"92330254", "statement.timestamp": { "$gt":"2016-03-10T00:00:00-00:00", "$lt":"2016-04-07T21:12:36.130Z" } } }, { "$group":{ "_id":"$statement.verb.id", "count": { "$sum":1 } } } ]

Sample Result:

{"result":[{"_id":"http://id.tincanapi.com/verb/viewed","count":183},{"_id":"https://brindlewaye.com/xAPITerms/verbs/loggedin/","count":53}],"ok":1,"$gleStats":{"lastOpTime":{"sec":0,"inc":0},"electionId":{"$id":"56e251b9437d1a2ccfac1b59"}}}



