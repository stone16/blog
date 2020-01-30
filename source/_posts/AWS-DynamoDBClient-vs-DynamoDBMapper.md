---
title: AWS DynamoDBClient vs DynamoDBMapper
date: 2020-01-29 20:09:13
categories: Cloud
tags:
    - AWS
    - DynamoDB
    - Mapper
top:
---
Recently just did a project related with DynamoDB, use both DynamoDBClient and DynamoDBMapper in different circumstances. In this post, will compare those two, regarding with its convenience, latency, and their differences by nature. 

# 1. Introduction 

DynamoDBClient is Service client for accessing DynamoDB; while DynamoDBMapper use ORM (Object relational mapping) for converting data between incompatible type systems using object-oriented programming languages. 

## 1.1 How to use DDBClient

    // Create POJO
    @Data
    public class Whitelist {
        private String id;
        private String content;
        private String status;
    }
    
    // Create DDBClient
    AmazonDynamoDB ddbCLient = AmazonDynamoDBClientBuilder.standard()
        .withCredentials(new AWSCredentialsProvider(KEY))
        .withRegion("aaabbb")
        .build();
        
    // Convert whitelist into Map
    Map<String, AttributeValue> whitelistMap = new HashMap<>();
    whitelistMap.put("id", "123");
    whitelistMap.put("content", "hello world");
    whitelistMap.put("status", "onSale");

    // UpdateItem Request
    UpdateItemRequest request = new UpdateItemRequest()
        .withTableName("WhitelistTable")
        .withItem(whitelistMap);
    
    // Use updateItem operation
    ddbClient.updateItem(request);

As you can see here, we need to build a String, AttributeValue map, which is a bit annoying. 

## 1.2 How to use DynamoDBMapper

We can define the table schema when we define the POJO using DynamoDBMapper's annotations. 

    // Notice: lombok may not work well here, especially when you use GSI and LSI, it cannot retrive info correctly
    
    @DynamoDBTable(tableName = "whitelistTable")
    public class Whitelist {
        
        @DynamoDBHashKey
        private String id;
        
        @DynamoDBRangeKey
        private String status;
        
        @DynamoDBAttribute
        private String content;
    }
    
    // Create DDBClient
    AmazonDynamoDB ddbCLient = AmazonDynamoDBClientBuilder.standard()
        .withCredentials(new AWSCredentialsProvider(KEY))
        .withRegion("aaabbb")
        .build();
        
    //Create DDBMapper
    AmazonDynamoDBMapper ddbMapper = new AmazonDynamoDBMapper(ddbClient);
    
    // updateItem
    ddbMapper.save(whitelistItem);
    
Code looks much more succinct, isn't it! As you can see above, you can think DDBMapper wraps DDBCLient in some way, and help you implement the String-AttributeValue map for you. 

# 2. Comparision 

+ DynamoDBMapper 
    + Benefits
        + Code look better
        + less dev work
    + weakness
        + it runs slower than using DDBClient
            + E.G when using DDBClient::getItem, the average latency is about 2ms; whereas it comes to be around 8ms in DDBMapper.  
+ DynamoDBClient
    + Benefits
        + As said above, run much faster
    + weakness
        + verbose code
        + easier for mistakes, typos 

So how to choose from those two -- it depends. Suppose you have a daily job to put 100 millions items into one table and want to write faster, DDBClient should definitely be your choice in this situation. Otherwise, use DynamoDBMapper to save your life, lol. 