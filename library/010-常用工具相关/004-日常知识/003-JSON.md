# Jackson JSON

## Use Jackson to parse Json String[1]

If you have a JSON String and you want to parse it to List of Object, then you can write as follow:

```shell
ObjectMapper mapper = new ObjectMapper();
List<MyObject> objects = mapper.readValue(jsonString, new TypeReference<List<MyObject>>(){});
```

or:

```
List<MyObjects> objects = mapper.readValue(jsonString, mapper.getTypeFactory().constructCollectionType(List.class, MyObject.class));
```

## Timestamp[2]

If your class have fields of type Timestamp ,you will failed when you want to parse json string to object. Then you can add annotations to fields of type Timestamp. Just like:

```java
@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss.SSS")
private Timestamp time;
```



## Reference

1.  https://stackoverflow.com/questions/6349421/how-to-use-jackson-to-deserialise-an-array-of-objects 
2.  https://stackoverflow.com/questions/43373270/jackson-deserialize-json-with-timestamp-field 