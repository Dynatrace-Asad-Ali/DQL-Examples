# DQL-Examples
## Purpose
This repo contains DQL examples to parse, filter and summarize data from the logs. The intention of this repo is for education and learning purpose only. Some of the DQL may not be fully optimized. All the DQL queries used here have been tested against demo.live environment. 

## Contribution
This is a work in progress repo. If you have examples that you wish to share, please make a pull request.

### Note
For some of the queries, a fieldsAdd is added before the query. This is done to give an example of the format of the content field. Since these queries are tested against the demo env which in turn can have different data at different times, the filtering added to the query may not bring the exact content that is required for the query to work. In that case, add the fieldsAdd section to your query to see it work.

## Examples

### Search for INFO and WARN messages.

This query uses the in(<needle>, <haystack>) command

```
fetch logs
| filter in(loglevel, array("INFO", "WARN"))
```

### Calculate the age in seconds and minutes of the most recent entry in the log file

```
fetch logs, from:now() -6h
| filter loglevel == "ERROR"
| limit 1
| fieldsAdd age_seconds = (now()-timestamp)/1000000000
| fieldsAdd age_minutes = age_seconds / 60
| fields timestamp, age_seconds , age_minutes 
```

### Filter data based upon event type and then summarize the results
```
fetch bizevents
| filter event.type == "com.easytrade.deposit-money"
| fields cardType, amount
| summarize sum(toDouble( amount)), by:{cardType}
```

### Summarizing quantity by products in the log file

Optional: [fieldsadd 2](https://github.com/Dynatrace-Asad-Ali/DQL-Examples/blob/main/optional/fieldsAdd.md)

```
fetch logs, from:now()-60m
| filter dt.process.name == "HipsterShop: cartservice"
| filter contains(content, "AddItemAsync")
| parse content, "LD:text 'productId=' alnum:product ', ' 'quantity=' int:qty"
| fields content, product, qty
// Summarize the qty by product
| summarize count=count(), sum(qty), by:{product}
```

### Filter the data and then parse the JSON within the content

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

### Parsing log file that has an array of array of JSON

```
fetch logs, from:now()-60m
| filter k8s.namespace.name == "easytrade"
| filter dt.process.name == "npm offerservice-*"
| filter contains(content, "Response:")
| parse content, "punct{1,1}:p1 timestamp('yyyy-MM-dd HH:mm:ss'):t punct{1,1}:p2 LD:text 'Response: ' string:pj"
| parse pj, "JSON_array:pj1"
| parse pj1[0][0], "JSON:pj1_0"
| fields content,pj1_0[id], pj1_0[name]
```

### Parse the content for http response code and count the number of times the http request has succeeded or failed
```
fetch logs, from:now()-3h
| filter contains(content, "POST /cart/checkout")
| filter dt.process.name != "HipsterShop: loadgenerator"
| parse content, "IPADDR:ip LD 'HTTP/1.1\"' SPACE int:http_r"
| fields timestamp, content, ip,http_r
| summarize success = countIf(http_r >= 200 and http_r <=400), fail = countIf(http_r >400), by:{ip, bin(timestamp, 10m)}
```

### Concat 2 strings to create a new field

The query parses the content for currency and the amount. The amount is split into 2 fields. The query concat the two strings to make a new field called totalUnit. The query can then be extended to create a bin of 100 for the totalUnit.

```
fetch logs, from:now()-5d
| filter contains(dt.process.name, "payment")
| filter loglevel == "INFO"
| filter contains (content, "credit_card")
| parse content, "JSON:pj"
| fields content, msg=pj[message]
| parse msg, "LD 'currency_code' punct{2,2} string:currency LD:text 'units' punct{3,3} int:unit LD 'nanos' punct{2,2} long:nanos"
| fieldsAdd totalUnitString = concat( unit, ".", nanos)
| fieldsAdd totalUnit = toDouble( totalUnitString)
| fields msg,currency, totalUnit 
// Additonal exercise
| summarize count(), by:{bin(totalUnit, 100)}
```

### Parse JSON content without using the JSON keyword

```
fetch logs, from:now()-5d
| filter contains(dt.process.name, "com.dynatrace.easytravel.business.backend.jar")
| filter loglevel == "INFO"
| parse content, "LD 'eventType' punct{3,3} [A-Z _]*:event"
| filter isNotNull( event)
| fields content, event
```

### Parse JSON content and extract embedded element in the JSON

```
fetch logs, from:now()-5d
| filter contains(dt.process.name, "com.dynatrace.easytravel.business.backend.jar")
| filter loglevel == "INFO"
| filter contains(content, "eventType")
| parse content, "LD 'body of: ' JSON:pj"
| fields content, custom=pj[customProperties][Owner]
```

### An example of using the KVP keyword

```
fetch logs, from:now()-5d
| filter dt.process.name == "hipstershop.AdService adservice-*"
| parse content, "LD 'instant' punct{3,3} LD:parsed"
| parse parsed, "KVP{punct{1,1} [a-zA-Z]*:key punct{2,2} [0-9]*:value ',' ?}:instantData"
| fields instantData[epochSecond]
```

### Parsing entries in the log file and returning true only if StringA and StringB did not occur at the same time in the same line

```

fetch logs
| fieldsAdd logData = "StringA Blah Blah StringA Blah Blah"
| fields logData
| limit 1
| fieldsAdd aString = contains(logData, "StringA"), bString = contains(logData, "StringB")
| fields logData , onlyOneExist = toDouble(aString) + toDouble( bString)
| filter onlyOneExist == 1
```

### Summarize log count by the day of the week.

```
fetch logs
| summarize logcount=count(), by:{ day = toLong(formatTimestamp(timestamp,format:"d"))}
| filter day < 8
| fieldsAdd Day=if(day==1,"Monday",else:if(day==2,"Tuesday",else:if(day==3,"Wednesday",else:if(day==4,"Thursday",else:if(day==5,"Friday",else:if(day==6,"Saturday",else:"Sunday"))))))
| fields logcount,Day
```

### Summarize log count by month

```
fetch logs, from:now()-60d
| summarize count(), by:{ month = formatTimestamp(timestamp,format:"YYYY-MM")}
```