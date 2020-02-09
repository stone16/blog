---
title: XML与JSON的比较
date: 2020-02-08 22:25:58
categories: BackEnd
tags:
    - BackEnd
top:
---
# 1. JSON

JSON - JavaScript Object Notation, an open standard file format uses human-readable text to transmit data objects consisting of attribute–value pairs and array data types (or any other serializable value). It is a very common data format used for asynchronous browser–server communication, including as a replacement for XML in some AJAX-style systems.

The official Internet media type for JSON is **application/json**. 

# 2. XML 

JSON is promoted as a **low-overhead alternative** to XML as both of these formats have widespread support for creation, reading, and decoding in the real-world situations where they are commonly used.

XML has been used to **describe structured data and to serialize objects**. Various XML-based protocols exist to represent the same kind of data structures as JSON for the same kind of data interchange purposes. Data can be encoded in XML in several ways. The most expansive form **using tag pairs results** in a much larger representation than JSON, but if data is stored in attributes and 'short tag' form where the closing tag is replaced with '/>', the representation is often about the same size as JSON or just a little larger. If the data is compressed **using an algorithm like gzip**, there is little difference because compression is good at saving space when a pattern is repeated.

XML also has the concept of **++schema++**. **This permits strong typing, user-defined types, predefined tags, and formal structure, allowing for formal validation of an XML stream in a portable way**. Similarly, there is an IETF draft proposal for a schema system for JSON.[44]

XML supports comments, but JSON does not

# 3. Differences

# 3.1 XML is a markup language whereas JSON is a way of representing objects

A markup language is a way of adding extra information to free-flowing plain text 

    <Document>
        <Paragraph Align="Center">
            Here <Bold>is</Bold> some text.
        </Paragraph>
    </Document>

An object notation like JSON is not as flexible. But this is usually a good thing. When you're representing objects, you simply don't need the extra flexibility. To represent the above example in JSON, you'd actually have to solve some problems manually that XML solves for you.

    {
        "Paragraphs": [
            {
                "align": "center",
                "content": [
                    "Here ", {
                        "style" : "bold",
                        "content": [ "is" ]
                    },
                    " some text."
                ]
            }
        ]
    }
    

JSON is better suited if you have typical a hierarchy of objects and you want to represent them in a stream. 

    {
        "firstName": "Homer",
        "lastName": "Simpson",
        "relatives": [ "Grandpa", "Marge", "The Boy", "Lisa", "I think that's all of them" ]
    } 
    
Below is same expression in xml

    <Person>
        <FirstName>Homer</FirstName>
        <LastName>Simpsons</LastName>
        <Relatives>
            <Relative>Grandpa</Relative>
            <Relative>Marge</Relative>
            <Relative>The Boy</Relative>
            <Relative>Lisa</Relative>
            <Relative>I think that's all of them</Relative>
        </Relatives>
    </Person>
    
## 3.2 JSON has defined ways of distinguishing records 

You can differenciate records directly in JSON. List and normal record have different expression; wheareas in XML, they look all same. 

We need to use an external schema or extra user defined attributes in XML to express different expressions, or some limitations on it. While in JSON, it's self describing by default. 

-> JSON should be the first choise for object notation, where XML should spot at document markup. 