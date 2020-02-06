---
title: Javadoc tutorial
date: 2020-02-05 19:04:12
categories: BackEnd
tags:
    - Java
top:
---
# 1. What is javadoc

A tool comes with JDK and used for generating Java code documentation in **HTML format from Java source code**, which requires documentation in a predefined format 

# 2. Javadoc tags

+ @author
+ {@code}
    + Displays text in code font without interpreting the text as HTML markup or nested javadoc tags.
+ {@docRoot}
    + Represents the relative path to the generated document's root directory from any generated page.
+ @deprecated
+ @exception
    + Adds a Throws subheading to the generated documentation, with the classname and description text.
+ {@inheritDoc}
    + Inherits a comment from the nearest inheritable class or implementable interface.
+ {@link}
    + Inserts an in-line link with the visible text label that points to the documentation for the specified package, class, or member name of a referenced class. 
+ @param
+ @return
+ @see
+ @serial
+ @serialData 
    + Documents the data written by the writeObject( ) or writeExternal( ) methods.
+ @serialField
    + Documents an ObjectStreamField component.
+ @since 
    + Adds a since heading with the specified since text to the generated documentation
+ @throws 
+ {@value}
    + When {@value} is used in the doc comment of a static field, it displays the value of that constant. 