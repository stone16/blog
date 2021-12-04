---
title: GraphQL Read
date: 2021-12-04 17:29:50
categories:
    - Web
tags:
top:
---
# GraphQL Read

# 1. Introduction

- QraphQL is
    - a query language
    - a server side runtime for executing queries using a type system you define

# 2. Queries and Mutations

- Fields
    - GraphQL is about asking specific fields on objects
    - Query has the same shape as result
        - server knows exactly what fields the client is asking for

```ruby
{
	hero {
		name
	}
}

{
	"data": {
		"hero": {
			"name": "12test"
		}
	}
}
```

- Arguments
    - we could pass arguments to fields
    - comparing with Restful, in GraphQL every field and nested object can get its own set of arguments, making GraphQL a complete replacement for making multiple APU fetches
- Fragments
    - That's the reusable units in GraphQL
    - Fragments let you construct sets of fields, and then include them in queries where you need to
    - It's commonly used to split complicated application data requirements into smaller chunks

- Operation Name
    - Operation Type
        - Query
        - Mutation
        - Subscription
    - Operation Name
- Variables
    - It want to give dynamic power to graphql, as in most applications, the arguments to fields will be dynamic
    - Graphql supports this use case via variables
    - we could do:
        - replace the static value in the query with `$variable`
        - declare `$variable` as one of the variables accepted by the query
        - pass `variable: value` in the separate transport specific variables dictionary
    - using variable could help us denote which arguments are expected to be dynamic
    - we should never do string interpolation to construct queries from user supplied values

```ruby
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}

{
  "episode": "JEDI"
} 
```

- Default variables

```ruby
query HeroNameAndFriends($episode: Episode = "defaultOne") {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}

{
  "episode": "JEDI"
} 
```

- Directives
    - use this variable to dynamically change the structure and shape of our queries using variables
    - `@include(if: Boolean)` only includes this field in the result if the argument is true
    - `@skip(if: Boolean)` skip this field if the argument is true
- Mutations
    - A way to modify server side data
    - A convention that any operations that cause writes should be sent explicitly via a mutation
    - !!! While query fields are executed in parallel, mutation fields run in series, one after the other
        - means if we send two incrementCredits mutations in one request, the first is guranteed to finish before the second begins, ensuring that we don't end up with a race condition with ourselves
- Inline Fragments
    - GraphQL schemas include the ability to define interfaces and union types
    - EG below, we need to return different attributes based on hero character
    
    ```ruby
    query HeroForEpisode($ep: Episode!) {
      hero(episode: $ep) {
        name
        ... on Droid {
          primaryFunction
        }
        ... on Human {
          height
        }
      }
    }
    ```
    
- Meta fields
    - there are situations where you don't know what type you'll get back from the service
    - we need to determine how to handle that data on the client
    - we could use `__typename`

# 3. Schemas and Types

## 3.1 how the schema work

- How does GraphQL work
    - start with a `root` object
    - select the hero field on that
    - for the object returned by hero, we select the name and appearsIn fields
    
    ```ruby
    {
      hero {
        name
        appearsIn
      }
    }
    ```
    
- we should know what we could query for
    - an exact description of the data we can ask for
    - what kind of objects might they return
    - what fields are available on those sub objects
- Schema
    - Each graphQL services defines a set of types which completely describe the set of possible data you can query on the service

## 3.2 Type Language

- GraphQL use its won Schema Language

### 3.2.1 Object Types and Fields

- Object types
    - represent a kind of object you can fetch from your service, and what fields it has

```ruby
type Character {
  name: String!
// means an array of Episode objects, notnull, 0 or more items, and each item would be an episode object 
  appearsIn: [Episode!]!
}
```

### 3.2.2 Query and Mutation types

- Entry points into the schema

### 3.2.3 Scalar Types

- Scalar types represent the leaves of the query
- default scalar types
    - Int
    - Float
    - String
    - Boolean
    - ID
        - it represents a unique identifier
        - The ID type is serialized in the same way as a String, but it means it's not intended to be human readable
- We could also define our own Scalar type in this way
    - `scalar Date`

### 3.2.4 Enumeration Types

- Restricted to a particular set of allowed values
- Allow you to
    - validate that any arguments of this type are one of the allowed values
    - communicate through the type system that a field will always be one of a finite set of values

### 3.2.5 Lists and Non-Null

```
type Character {
  name: String!
  appearsIn: [Episode]!
}

```

- We could use `!` to indicate it should never return null
- We could use `[]` to indicate that should be an array

### 3.2.6 Interfaces

- An abstract type that includes a certain set of fields that a type must include to implement the interface

```ruby
interface Character {
	id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}

type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}

type Droid implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  primaryFunction: String
}
```

- Type implement the interface need to have all those fields, but they could also have their own fields

### 3.2.7 Union Types

``union SearchResult = Human | Droid | Starship``

### 3.2.8 Input Types

- we need to pass complex objects especially when we are using mutations, where we want to pass in a whole object to be created

```ruby
input ReviewInput {
  stars: Int!
  commentary: String
}

mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

# 4. Validation

- Graph ql has validation module to fulfill the validation phase of fulfilling a graphQL result

# 5. Execution

## 5.1 Resolvers

- Each field in a GraphQL query is a function or method of the previous type which returns the next type
- Each field is backed by a function called the resolver. When a field is executed, the corresponding resolver is called to produce the next value
- The resolver continue to work until reach scalar values

## 5.2 Root fields

```ruby
Query: {
  human(obj, args, context, info) {
    return context.db.loadHumanByID(args.id).then(
      userData => new Human(userData)
    )
  }
}
```

- obj
    - previous object
- args
    - arguments provided to the field in the graphQL query
- context
    - a value which is provided to every resolver and holds important contextual information like the currently logged in user, or access to a database
- info
    - a value which holds field specific information relevant to the current query as well as the schema details

# 6. Best Practices

## 6.1 HTTP

- Mostly use HTTP with graphQL
- Normally web frameworks use a pipeline model where requests are passed through a stack of middle ware
- Requests could be inspected, transformed, modified or terminated with a response
- GraphQL should be placed after all authentication middleware— thus you have access to the same session and user info you would in your HTTP endpoints handler

- GraphQL server operates on a single URL/ endpoint, usually `graphql` , and all graphql requests for a given service should be directed at this endpoint

## 6.2 JSON with GZIP

- typically respond using JSON, and we compress it with GZIP

## 6.3 Versioning

- No need to do versioning for graphql api
- Why do most APIs version? When there's limited control over the data that's returned from an API endpoint, *any change* can be considered a breaking change, and breaking changes require a new version. If adding new features to an API requires a new version, then a tradeoff emerges between releasing often and having many incremental versions versus the understandability and maintainability of the API.
- In contrast, GraphQL only returns the data that's explicitly requested, so new capabilities can be added via new types and new fields on those types without creating a breaking change. This has led to a common practice of always avoiding breaking changes and serving a versionless API.

## 6.4 Nullability

- GraphQL default to nullable unless you specifically declare nonnull

# Reference

1. [https://www.graphql-java-kickstart.com/](https://www.graphql-java-kickstart.com/)  
2. [https://graphql.org/learn/](https://graphql.org/learn/)  
3. [https://www.apollographql.com/docs/federation/](https://www.apollographql.com/docs/federation/)