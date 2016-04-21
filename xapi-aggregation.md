#Querying xAPI Activity Data

Whilst xAPI provides basic ways of querying data, this has not been sufficently flexible to provide access to data for most applications integrated as part of the Jisc project.  We therefore recommend using the Learning Locker Aggregration API, which is based on MongoDB pipelines.  

It is documented here: http://docs.learninglocker.net/statements_api/

For the Jisc learning locker, the base URL for aggregation is 

http://jisc.learninglocker.net/api/v1/statements/aggregate?pipeline=[]

and then pipelines can be added as JSON array elements between the square brackets.

Each pipeline takes the form { "$\<pipeline_keyword\>": {\<pipeline statements\>} }

Pipelines are processed in order, and the result set from each pipeline is passed to the next one. For this reason, usually the first pipeline is a $match pipeline to filter a smaller result set, and then aggregations like $group can be used in subsequent stages.

## Building understanding and getting to know the data

Filtering and aggregating relies on an understanding of the data structure. You can of course look through a sample dataset offline, but here's a walkthrough of how to play with the endpoint itself in order to inform more detailed queries. 

First, let's assume you know nothing about the structure of xAPI. You could just hit the endpoint with an empty query and pull all the records, but let's assume you want to be nice to the other people in the Hackathon. You can just pull the first record in the database instead to have a look at its structure:

{"$match": {}}, {"$limit": 1}

with sample result:

{
  "result": [
    {
      "_id": {
        "$id": "571132b5058ebcc0258b5b2f"
      },
      "lrs": {
        "_id": {
          "$id": "56de73ac058ebc77698bb407"
        }
      },
      "lrs_id": {
        "$id": "56de73ac058ebc77698bb407"
      },
      "client_id": "56de73ac058ebc77698bb408",
      "statement": {
        "version": "1.0.1",
        "timestamp": "2016-03-31T20:29:17-07:00",
        "object": {
          "objectType": "Activity",
          "definition": {
            "extensions": {
              "http://xapi.jisc.ac.uk/extensions/applicationType": {
                "type": "http://xapi.jisc.ac.uk/define/vle"
              }
            },
            "type": "http://activitystrea.ms/schema/1.0/application",
            "description": {
              "en": "University of Jisc Moodle ;"
            },
            "name": {
              "en": "University of Jisc Moodle "
            }
          },
          "id": "http://moodle.data.alpha.jisc.ac.uk/course/view.php?id=1"
        },
        "actor": {
          "objectType": "Agent",
          "account": {
            "homePage": "http://moodle.data.alpha.jisc.ac.uk/",
            "name": "90006516"
          },
          "name": "90006516"
        },
        "verb": {
          "id": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "display": {
            "en": "logged in to"
          }
        },
        "context": {
          "platform": "Moodle",
          "extensions": {
            "http://id.tincanapi.com/extension/ip-address": {
              "ip-address": "0.0.0.1"
            },
            "http://lrs.learninglocker.net/define/extensions/info": "University of Jisc Moodle",
            "http://xapi.jisc.ac.uk/extensions/sessionId": {
              "sessionId": "na"
            }
          }
        },
        "authority": {
          "objectType": "Agent",
          "name": "Michael Webb",
          "mbox": "mailto:michael.webb@jisc.ac.uk"
        },
        "stored": "2016-04-15T18:28:05.615800+00:00",
        "id": "7d74e309-f768-42e1-be8a-dac991f217d5"
      },
      "active": true,
      "voided": false,
      "timestamp": {
        "sec": 1459481357,
        "usec": 0
      },
      "stored": {
        "sec": 1460744885,
        "usec": 615000
      }
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 1461227571,
      "inc": 15
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

You'll see that the record comes back as JSON, with an array 'result' and some stats at the end. The result array has all the associated metadata mixed in for each result element, but if you look down you'll see a 'statement' object which contains the xAPI statement. Inside there you'll see the actor/verb/object relationship as well as a context element. Hopefully a lot of the fields will be self-explanatory!

You can match and aggregate any of the fields you see in the JSON response. Again, though, for the sake of performance, as well as being nice to the server and your fellow participants, you should probably get into the mindset of aggregating responses where possible unless you know you require full records, and you know your query won't pull large numbers of records.

With that in mind, here's an aggregation query to find out how many distinct actors there are in the data using two group pipelines -- the first to isolate the distinct actor names, and the second to count them:

{"$match": {}}, {"$group": {"_id": "$statement.actor.name"} }, {"$group": {"_id": "actors", "count": {"$sum": 1}} }

with sample result:

{
  "result": [
    {
      "_id": "actors",
      "count": 357
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 1461249803,
      "inc": 7
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

357 doesn't seem like *too* many, so let's go ahead and run a simpler aggregation query just returning the names of all the actors in the data, along with counts of their statements. For this query, we don't want to filter the data at all, so the match is empty. 

{"$match": {}}, {"$group": {"_id": "$statement.actor.name", "count": {"$sum":1} } }

with sample result:

{
  "result": [
    {
      "_id": "92692514",
      "count": 5331
    },
    {
      "_id": "92688166",
      "count": 847
    },
    
    ... (357 records total)
  
  ]
}

Similarly, we might be interested in all the different verbs which are in use in the dataset -- here's a query to bring back a list of all of them with counts. This time I have pulled the entire verb object instead of just one field, so you can see both the verb id URI and the english description of it:

{"$match": {}}, {"$group": {"_id": "$statement.verb", "count": {"$sum":1} } }

with sample result:

{
  "result": [
    {
      "_id": {
        "id": "http://id.tincanapi.com/verb/viewed",
        "display": {
          "en": "viewed"
        }
      },
      "count": 440848
    },
    {
      "_id": {
        "id": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
        "display": {
          "en": "logged in to"
        }
      },
      "count": 92640
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 0,
      "inc": 0
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

## Sample Queries

From this point forward, the examples are real examples used in the Jisc student app!

### Activity by actor

If you were only interested in the 'activity' (verb URIs and counts) for a particular actor, then the match filter would come in:        (internal Jisc note: from STUAPP 5)

{"$match": {"statement.actor.name":"92330254"} }, 
{"$group": {"_id": "$statement.verb.id", "count": {"$sum":1} } }

with sample result:

{
  "result": [
    {
      "_id": "http://id.tincanapi.com/verb/viewed",
      "count": 2512
    },
    {
      "_id": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
      "count": 540
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 0,
      "inc": 0
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

### Activity by actor within time period

Maybe you just want to see if that actor did anything in January, so you would bring in the concept of AND by using multiple filter terms and bringing in date filtering:        (internal Jisc note: from STUAPP 3)

{ "$match":{ "statement.actor.name":"92330254", "statement.timestamp": { "$gt":"2016-01-01T00:00:00-00:00", "$lt":"2016-02-01T00:00:00-00:00" } } }, { "$group":{ "_id":"$statement.verb.id", "count": { "$sum":1 } } }

with sample result:

{
  "result": [
    {
      "_id": "http://id.tincanapi.com/verb/viewed",
      "count": 624
    },
    {
      "_id": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
      "count": 131
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 0,
      "inc": 0
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

### Activity by actor on module

One particular filter of interest might be to only return activity on a certain module:        (internal Jisc note: from STUAPP 11)

{"$match":{ "statement.actor.name":"92330254", "statement.object.definition.name.en":"MODS101" }},
{"$group":{"_id":"$statement.verb.id","count":{ "$sum":1 }}}

with sample result:

{
  "result": [
    {
      "_id": "http://id.tincanapi.com/verb/viewed",
      "count": 978
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 0,
      "inc": 0
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

### VLE activity by actor

You might want to filter out everything except VLE generated data -- here we introduce the concept of OR by using an array of filter terms:        (internal Jisc note: from STUAPP 19)

{"$match":{"statement.actor.name":"92330254","statement.context.platform":{ "$in":[ "Blackboard", "Moodle" ] }}},
{"$group":{"_id":"vle","count":{ "$sum":1 }}}

with sample result:

{
  "result": [
    {
      "_id": "vle",
      "count": 3052
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 1461230765,
      "inc": 4
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

## Restructuring output data by adding multiple $group pipelines

Maybe you want to break down the aggregate, so that instead of using the above query for one month at a time, you issue one query and partition the results by month. To achieve this, we make use of a second group query at the end of the pipeline. Recall, each step of the pipeline passes its output to the next, so in this way you can also transform the structure of the returned dataset! 
The first group makes use of an id object to perform a more complicated 'GROUP BY' step -- grouping and counting statements where both the month (obtained from the first 7 characters of the timestamp string) and the verb are the same.
We then pass that data to a pipeline which makes use of the 'push' keyword to build a data structure on the fly, taking all verb/month pairs with the same value of month and bundling them up together:        (internal Jisc note: from STUAPP 7)

{"$match":{ "statement.actor.name":"92330254" }},
{"$group":{"_id":{"month":{ "$substr":[ "$statement.timestamp", 0, 7 ] },"verb":"$statement.verb.id"},"count":{ "$sum":1 }}},
{"$group":{"_id":"$_id.month","counts":{"$push":{ "verb":"$_id.verb", "count":"$count" }}}}

with sample result:

{
  "result": [
    {
      "_id": "2016-04",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 133
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 40
        }
      ]
    },
    {
      "_id": "2016-03",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 470
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 123
        }
      ]
    },
    {
      "_id": "2016-01",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 624
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 131
        }
      ]
    },
    {
      "_id": "2016-02",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 661
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 151
        }
      ]
    },
    {
      "_id": "2015-12",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 624
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 95
        }
      ]
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 1461230765,
      "inc": 4
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

### Grouped activity by month for actor

You can restrict the above query to a time period of interest:        (internal Jisc note: from STUAPP 8)

{"$match":{"statement.actor.name":"92330254" ,"statement.timestamp":{ "$gt":"2016-02-03T00:00:00-00:00"}}},
{"$group":{"_id":{"month":{ "$substr":[ "$statement.timestamp", 0, 7 ] },"verb":"$statement.verb.id"},"count":{ "$sum":1 }}},
{"$group":{"_id":"$_id.month","counts":{"$push":{ "verb":"$_id.verb", "count":"$count" }}}}

with sample result:

{
  "result": [
    {
      "_id": "2016-04",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 133
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 40
        }
      ]
    },
    {
      "_id": "2016-03",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 470
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 123
        }
      ]
    },
    {
      "_id": "2016-02",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 621
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 143
        }
      ]
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 0,
      "inc": 0
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

### Grouped activity for by actor from set 

If you're interested in comparing actors instead of months, you can return verbs for more than one using the array OR filtering:        (internal Jisc note: from STUAPP 2)

{"$match":{"statement.actor.name":{ "$in":[ "92330254", "92244332", "91576513", " 90998493", "90293824" ] }}},
{"$group":{"_id":{ "user":"$statement.actor.name", "verb":"$statement.verb.id" },"count":{ "$sum":1 }}},
{"$group":{"_id":"$_id.user","counts":{"$push":{ "verb":"$_id.verb", "count":"$count" }}}}

with sample result:

{
  "result": [
    {
      "_id": "91576513",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 8846
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 1125
        }
      ]
    },
    {
      "_id": "92244332",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 7747
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 1083
        }
      ]
    },
    {
      "_id": "92330254",
      "counts": [
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 540
        },
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 2512
        }
      ]
    },
    {
      "_id": "90293824",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 5803
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 924
        }
      ]
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 1461230765,
      "inc": 4
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

### Grouped activity by actor from set in a given time period

Again, you can develop the above query by restricting it to a time period of interest:        (internal Jisc note: from STUAPP 1)

{ "$match":{"statement.actor.name":{ "$in":[ "92330254", "92244332", "91576513", " 90998493", "90293824" ] }, "statement.timestamp": { "$gt":"2014-01-01T00:00:00-00:00" }} },
{"$group":{"_id":{ "user":"$statement.actor.name", "verb":"$statement.verb.id" },"count":{ "$sum":1 }}},
{"$group":{"_id":"$_id.user","counts":{"$push":{ "verb":"$_id.verb", "count":"$count" }}}}

with sample result:

{
  "result": [
    {
      "_id": "91576513",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 8846
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 1125
        }
      ]
    },
    {
      "_id": "92244332",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 7747
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 1083
        }
      ]
    },
    {
      "_id": "92330254",
      "counts": [
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 540
        },
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 2512
        }
      ]
    },
    {
      "_id": "90293824",
      "counts": [
        {
          "verb": "http://id.tincanapi.com/verb/viewed",
          "count": 5803
        },
        {
          "verb": "https://brindlewaye.com/xAPITerms/verbs/loggedin/",
          "count": 924
        }
      ]
    }
  ],
  "ok": 1,
  "$gleStats": {
    "lastOpTime": {
      "sec": 1461230765,
      "inc": 4
    },
    "electionId": {
      "$id": "56e251b9437d1a2ccfac1b59"
    }
  }
}

### Grouped activity by actor and day on a module

Finally, if you are feeling particularly perverse, you might want to build in more levels of hierarchy to your result set. Each layer you want to add will require an extra group pipeline. For example, this lollapalooza of all queries which groups up activity on a module by both actor and day!        (internal Jisc note: from STUAPP 12)

{"$match":{"statement.actor.name":{ "$in":[ "92330254", "92244332" ] },"statement.object.definition.name.en":"MODS101", "statement.timestamp":{ "$gt":"2014-10-01T00:00:00" }}},
{"$group":{"_id":{"student":"$statement.actor.name","day":{ "$substr":[ "$statement.timestamp", 0, 10 ] },"verb":"$statement.verb.id"},
"count":{ "$sum":1 }}},
{"$group":{"_id":{ "day":"$_id.day", "student":"$_id.student" },"counts":{"$push":{ "verb":"$_id.verb", "count":"$count" }}}},
{"$group":{"_id":"$_id.day","students":{"$push":{ "student":"$_id.student", "counts":"$counts" }}}}

with sample result:

{
  "result": [
    {
      "_id": "2016-01-05",
      "students": [
        {
          "student": "92330254",
          "counts": [
            {
              "verb": "http://id.tincanapi.com/verb/viewed",
              "count": 15
            }
          ]
        },
        {
          "student": "92244332",
          "counts": [
            {
              "verb": "http://id.tincanapi.com/verb/viewed",
              "count": 17
            }
          ]
        }
      ]
    },
    {
      "_id": "2016-01-24",
      "students": [
        {
          "student": "92244332",
          "counts": [
            {
              "verb": "http://id.tincanapi.com/verb/viewed",
              "count": 34
            }
          ]
        },
        {
          "student": "92330254",
          "counts": [
            {
              "verb": "http://id.tincanapi.com/verb/viewed",
              "count": 8
            }
          ]
        }
      ]
    },
    
    ... (125 blocks in total)
    
  ]
}  

### To infinity, and beyond

Following the idea here, you might want to satisfy yourself as an exercise that you could go one step further and group up by module, actor and day -- if you can achieve that, then you've truly mastered my little tutorial. Happy aggregating!
