# DQL-Examples
## Purpose
This repo contains DQL examples to parse, filter and summarize data from the logs. The intention of this repo is for education and learning purpose only. Some of the DQL may not be fully optimized. All the DQL queries used here have been tested against demo.live environment. 

## Contribution
This is a work in progress repo. If you have examples that you wish to share, please make a pull request.

### Note
For some of the queries, a fieldsAdd is added before the query. This is done to give an example of the format of the content field. Since these queries are tested against the demo env which in turn can have different data at different times, the filtering added to the query may not bring the exact content that is required for the query to work. In that case, add the fieldsAdd section to your query to see it work.

## Examples

### Filter data based upon event type and then summarize the results
```
fetch bizevents
| filter event.type == "com.easytrade.deposit-money"
| fields cardType, amount
| summarize sum(toDouble( amount)), by:{cardType}
```

### Filter the data and then parse the JSON within the content

```
{
  "instant": {
    "epochSecond": 1663941471,
    "nanoOfSecond": 624155000
  },
  "thread": "grpc-default-executor-11305",
  "level": "INFO",
  "loggerName": "hipstershop.AdService",
  "message": "received ad request (context_words=[clothing, tops])",
  "endOfBatch": false,
  "loggerFqcn": "org.apache.logging.log4j.spi.AbstractLogger",
  "threadId": 12158,
  "threadPriority": 5,
  "logging.googleapis.com/trace": "00000000000000000000000000000000",
  "logging.googleapis.com/spanId": "0000000000000000",
  "logging.googleapis.com/traceSampled": "false",
  "time": "2022-09-23T13:57:51.624Z"
}
```

```
fetch logs, from:now()-3h
| filter contains(dt.process.name, "adservice")
| filter k8s.deployment.name == "adservice-*"
| parse content, "JSON:pj"
| fields content, msg=pj[message]
| parse msg, "LD:text 'context_words=[' [a-z, ]*:keyword"
| fields msg,text, keyword
| summarize count=count(), by:{keyword}
```