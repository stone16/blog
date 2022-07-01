---
title: 'PG::DuplicatePstatement Error prepared statement axxx already exists'
date: 2022-07-01 22:04:04
categories: Ruby
tags:
    - Postgres 
top:
---
That’s an interesting issue we faced in production during one deployment which only contain some frontend changes. 

Upon check, this happens in such scenario: 

```java
A prepared statement is generated in postgresql, but never stored in rails. 
Since the code was interrupted before storing the statement, the @counter variable 
was never incremented even though it was used to generate a prepared statement.
```

That pretty much described the issue, prepared statement on postgres side is a server side object that can be used to optimize performance. When the `PREPARE`
 statement is executed, the specified statement is parsed, analyzed, and rewritten. When an `EXECUTE`command is subsequently issued, the prepared statement is planned and executed.  

When the identifiers already bound to existing prepared statements but rails does not realize it, this issue will be happened. 

here is the fix 

[https://github.com/rails/rails/pull/41356/files](https://github.com/rails/rails/pull/41356/files)

```java
def next_key 
	"a#{@counter + 1}"
end 

def next_key 
	"a#{@counter += 1}"
end 

This change make the postgres prepared statement counter before makeing a prepared statement
Thus if the statemnt is aborted in rails side, app won't end up in perpetual crash state 
```

# Reference

1. https://github.com/rails/rails/issues/1627
2. https://github.com/rails/rails/pull/25827
3. https://github.com/rails/rails/pull/17607