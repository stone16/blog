---
title: Jackson  java-json parser
date: 2020-02-05 18:55:49
categories: BackEnd
tags:
    - Jackson
    - Java
    - Serialization
top:
---
# 1. Intro 

+ JSON 
    + JavaScript Object Notation 
    + Data exchange format between browsers and web servers 
+ Jackson 
    + 2 main parsers 
        + ObjectMapper 
            + parse JSON into custom Java objects, or into a jackson specific tree structure 
        + JsonParser 
            + JSON pull parser, parsing JSON one token at a time 
    + 2 main JSON generator
        + ObjectMapper 
        + JsonGenerator
            + generate JSON one token at a time 
    + 3 main packages 
        + Jackson Core
        + Jackson Annotations
        + Jackson Databind 


# 2. Parsers - JSON to Java Object 

    ObjectMapper objectMapper = new ObjectMapper();
    String lakers = "{ \"SuperStar\":\"Kobe Bryant\"}";
    
    try {
        Lakers lakers = objectMapper.readValue(lakers, Lakers.class);
    } catch (IOException e) {
        log.error(e);
    }

    @Data
    public class Lakers {
        private String superStar;
    }

## 2.1 How Jackson ObjectMapper matches JSON fields to Java Fields? 

By default, Jackson maps the fields of a JSON object to fields in a Java object by matching the names of the JSON field to the getter and setter methods in the Java object. 
Jackson removes the "get" and "set" part of the names of the getter and setter methods, and converts the first character of the remaining name to lowercase.

If you want to customize the parsing process, you may want to use a custom serializer and deserializer, or use Jackson Annotations 


## 2.2 Fail on Null for primitive types 

We could configure the Jackson ObjectMapper to fail if a JSON string contains a field with its value set to null. 

    ObjectMapper objectMapper = new ObjectMapper();
    
    objectMapper.configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, true);
    
    
## 2.3 Jackson JsonParser 

+ lower level than the ObjectMapper 
    + faster than the ObjectMapper

+ Create a JsonParser 
    + `JsonFactory factory = new JsonFactory();`
    + `JsonParser parser = factory.createParser(carJson);`

+ Parsing Json with JsonParser
    + break the JSON up into a sequence of tokens which you can iterate one by one 


    String carJson =
            "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";
    
    JsonFactory factory = new JsonFactory();
    JsonParser  parser  = factory.createParser(carJson);
    
    while(!parser.isClosed()){
        JsonToken jsonToken = parser.nextToken();
    
        System.out.println("jsonToken = " + jsonToken);
    }

# 3. Generators - Java Object to JSON 

+ ObjectMapper
    + writeValue()
    + writeValueAsString()
    + writeValueAsBytes()

## 3.1 Jackson JsonGenerator 

+ used to generate JSON from java objects 

    
    JsonFactory factory = new JsonFactory();
    
    JsonGenerator generator = factory.createGenerator(
        new File("blabla"), JsonEncoding.UTF8);


# 4. Jackson JSON Tree Model 

+ A built-in tree model: used to represent a JSON object 
+ Represented by the JsonNode class 
    + use the ObjectMapper to parse JSON into a JsonNode tree model
+ JsonNode class lets you navigate the JSOn as a Java object in a quite flexible and dynamic way 


    String carJson =
            "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";
    
    ObjectMapper objectMapper = new ObjectMapper();
    
    try {
    
        JsonNode jsonNode = objectMapper.readValue(carJson, JsonNode.class);
    
    } catch (IOException e) {
        e.printStackTrace();
    }

    // ways on how to access JSON fields, arrays and nested objects 
    String carJson =
        "{ \"brand\" : \"Mercedes\", \"doors\" : 5," +
        "  \"owners\" : [\"John\", \"Jack\", \"Jill\"]," +
        "  \"nestedObject\" : { \"field\" : \"value\" } }";

    ObjectMapper objectMapper = new ObjectMapper();
    
    try {
        JsonNode jsonNode = objectMapper.readValue(carJson, JsonNode.class);
    
        // we could always use get() to get the node 
        JsonNode brandNode = jsonNode.get("brand");
        String brand = brandNode.asText();
        System.out.println("brand = " + brand);
    
        JsonNode doorsNode = jsonNode.get("doors");
        int doors = doorsNode.asInt();
        System.out.println("doors = " + doors);
    
        JsonNode array = jsonNode.get("owners");
        JsonNode jsonNode = array.get(0);
        String john = jsonNode.asText();
        System.out.println("john  = " + john);
    
        JsonNode child = jsonNode.get("nestedObject");
        JsonNode childField = child.get("field");
        String field = childField.asText();
        System.out.println("field = " + field);
    
    } catch (IOException e) {
        e.printStackTrace();
    }
## 4.1 Read JsonNode from JSON 

    String jsonStr = blablabla;
    
    ObjectMapper objectMapper = new ObjectMapper();
    
    JsonNode jsonNode = objectMapper.readTree(json);
    
## 4.2 Write JsonNode to JSON 

    ObjectMapper objectMapper = new ObjectMapper();
    
    JsonNode jsonNode = readJsonIntoJsonNode();
    
    String json = objectMapper.writeValueAsString(jsonNode);

# 5. JsonAnnotation 

+ @JsonIgnore
+ @JsonIgnoreProperties
    + specify a list of properties of a class to ignore 
+ @JsonIgnoreType
+ @JsonAutoDetect 
+ @JsonSetter 
+ @JsonCreator
+ @JsonProperty 
+ @JsonInclude
    + tells Jackson only to include properties under certain circumstances
+ @JsonGetter 
    + tell Jackson that a certain field value should be obtained from calling a getter method instead of via direct field access 
+ @JsonPropertyOrder
    + specify in what order the fields of your java object should be serialized into JSON
+ @JsonValue
    + tells jackson that it should not attempt to serialize the obejct itself, but rather call a method on the object which serialize the object to a JSON string 
+ @JsonSerialize 
    + specify a custom serializer for a field in a Java object  

# Reference 

http://tutorials.jenkov.com/java-json/jackson-objectmapper.html