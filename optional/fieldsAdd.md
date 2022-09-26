
### Optional FieldsAdd 1

```
| fieldsAdd content = "{ \"instant\": { \"epochSecond\": 1663941471, \"nanoOfSecond\": 624155000 }, \"thread\": \"grpc-default-executor-11305\", \"level\": \"INFO\", \"loggerName\": \"hipstershop.AdService\", \"message\": \"received ad request (context_words=[clothing, tops])\", \"endOfBatch\": false, \"loggerFqcn\": \"org.apache.logging.log4j.spi.AbstractLogger\", \"threadId\": 12158, \"threadPriority\": 5, \"logging.googleapis.com/trace\": \"00000000000000000000000000000000\", \"logging.googleapis.com/spanId\": \"0000000000000000\", \"logging.googleapis.com/traceSampled\": \"false\", \"time\": \"2022-09-23T13:57:51.624Z\" }"
```

### Option FieldsAdd 2

```
| fieldsAdd content="AddItemAsync called with userId=51f1ee80-09d8-49a6-8108-ecc99e58e689, productId=0PUK6V6EV0, quantity=4"
```

```
| fieldsAdd content = "[2022-09-23 15:36:33] [DEBUG]: Response: [[{\"id\":1,\"name\":\"Starter\",\"price\":0,\"support\":\"['Email']\"},{\"id\":2,\"name\":\"Light\",\"price\":24.99,\"support\":\"['Email','Hotline']\"},{\"id\":3,\"name\":\"Pro\",\"price\":49.99,\"support\":\"['Email','Hotline','AccountManager']\"}]]"
```