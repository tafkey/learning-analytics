#Sample xAPI Data Set

A sample xAPI data set is availble, with around 500k records, derived from the OU's Open Data. 

xAPI is in Jisc xAPI VLE recipe v0.1 RC1 format.

The file is split into files of around 100k records to make opening in text browser easier, and consists of comma seperated xAPI statement.

The data matches the sample student data.

[Download xAPI data Set](xapi-test-set.zip)

##Sample row
``` JSON
{  
   "actor":{  
      "name":"92358969",
      "account":{  
         "homePage":"http://moodle.data.alpha.jisc.ac.uk/",
         "name":"92358969"
      }
   },
   "context":{  
      "platform":"Moodle",
      "extensions":{  
         "http://xapi.jisc.ac.uk/extensions/sessionId":{  
            "sessionId":"na"
         },
         "http://id.tincanapi.com/extension/ip-address":{  
            "ip-address":"0.0.0.1"
         },
         "http://lrs.learninglocker.net/define/extensions/info":"University of Jisc Moodle",
         "http://xapi.jisc.ac.uk/extensions/courseArea":{  
            "id":"http://moodle.data.alpha.jisc.ac.uk/course/view.php?id=3",
            "http://xapi.jisc.ac.uk/extensions/vle_mod_id":"MODS101"
         }
      }
   },
   "timestamp":"2016-04-08T17:20:00-07:00",
   "verb":{  
      "id":"http://id.tincanapi.com/verb/viewed",
      "display":{  
         "en":"viewed"
      }
   },
   "object":{  
      "id":"http://moodle.data.alpha.jisc.ac.uk/mod/page/view.php?id=3",
      "definition":{  
         "type":"http://xapi.jisc.ac.uk/define/vle/page",
         "name":{  
            "en":"More Detail Page"
         },
         "description":{  
            "en":"<p>More detail</p>"
         },
         "http://xapi.jisc.ac.uk/extensions/applicationType":{  
            "type":"http://xapi.jisc.ac.uk/define/vle"
         }
      }
   }
},


``` JSON
