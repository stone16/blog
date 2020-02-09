---
title: JSP Tutorial
date: 2020-02-08 21:16:31
categories: BackEnd
tags:
    - Java
    - JSP
top:
---
# 1. Introduction 
## 1.1 Intro
+ JavaServer Pages 
    + develop webpages that support dynamic content 
    + collect input from users through webpage forms
    + present records from a database or another source 
    + creates webpage dynamically 

## 1.2 Architecture

+ JSP engine
    + a container to process JSP pages  
    + responsible for intercepting requests for JSP pages 

+ JSP processing 
    + browser sends an HTTP request to the web server 
    + web server recognizes that the HTTP request is for a JSP page and forwards it to a JSP engine. 
        + Finish by using the URL or JSP page which ends with .jsp instead of .html 
    + JSP engine loads the JSP page from disk and converts it into a servlet content. 
    + JSP engine compiles the servlet into an executable class and forwards the original request to a servlet engine
    + A part of the web server called the servlet engine loads the Servlet class and executes it. During execution, the servlet produces an output in HTML format. The output is furthur passed on to the web server by the servlet engine inside an HTTP response.
    + The web server forwards the HTTP response to your browser in terms of static HTML content !!! Return static html directly 
    + the web browser handles the dynamically generated HTML page inside the HTTP response exactly as if it were a static page 

## 1.3 Lifecycle 

A JSP life cycle is defined as the process from its creation till the destruction, Similar to a servlet life cycle with an additional step which is required to compile a JSP into servlet.

+ compilation
    + when a browser asks for a JSP, the JSP engine first checks to see whether it needs to compile the page 
    + If the page has never been compiled, or if the JSP has been modified since it was last compiled, the JSP engine compiles the page
    + compile involves:
        + parsing the JSP
        + turning the JSP into a servlet 
        + compile the servlet 
+ initialization
    + invokes the jspInit() method before servicing any requests  
+ execution
    + represents all interactions with requests until the JSP is destroyed 
    + Whenever a browser requests a JSP and the page has been loaded and initialized, the JSP engine invokes the _jspService() method in the JSP
    + `void _jspService(HttpServletRequest request, HttpServletResponse response) {
   // Service handling code...
}`
+ cleanup 
    + represents when a JSP is being removed from use by a container 
    + The jspDestroy() method is the JSP equivalent of the destroy method for servlets. 

# 2. Syntax/ Operations

## 2.1 Elements of JSP

### 2.1.1 Scriptlet

A scriptlet can contain any number of JAVA language statements, variable or method declarations, or expressions that are valid in the page scripting language. 

`<%code fragment%>`

XML equivalent as follows: 

    <jsp:scriptlet>
       code fragment
    </jsp:scriptlet>


Any text, HTML tags, or JSP elements you write must be outside the scriptlet. 

### 2.1.2 JSP Declarations

A declaration declares one or more variables or methods that you can use in Java code later in the JSP file. You must declare the variable or method before you use it in the JSP file.

    <%! declaration; [ declaration; ]+ ... %>
    
We can also write the XML equivalent of the above syntax as follows: 

    <jsp:declaration>
       code fragment
    </jsp:declaration>

### 2.1.3 JSP Expression 

+ Contains a scripting language expression that is evaluated, converted to a String, and inserted where the expression appears in the JSP file. 
+ The expression element can contain any expression that is valid according to the Java Language Specification but you cannot use a semicolon to end an expression.


    <%= expression %>
    
    <jsp:expression>
        expression
    </jsp:expression>

### 2.1.4 JSP Comments 

JSP comments marks text or statements that the JSP container should ignore. A JSP comment is useful when you want to hide or comment out. 


<%-- This is JSP comment --%>

### 2.1.5 JSP Directives 

+ A JSP directive affects the overall structure of the servlet class. 


    <%@ directive attribute="value" %>


+ `<%@ page. attribute=.. %>`
    + Defines page dependent attributes
        + scripting language
        + error page
        + buffering requirements 
    + instructions to the current page 
    + attributes list 
        + buffer 
        + autoFlush 
        + contentType
        + errorPage
        + isErrorPage
        + extends
        + import
        + info 
        + isThreadSafe
        + language
        + session 
        + isELignored 
        + isScriptingEnabled
+ `<%@ include ... %>`
    + include a file during the translation phase  
    + tells the container to merge the content of other external files with the current JSP during the translation phase.
+ `<%@ taglib ... %>`
    + declares a tag library, containing custom actions, used in the page  

### 2.1.6 JSP Actions 

JSP actions use contructs in XML syntax to control the behavior of the servlet engine

You can dynamically insert a file, reuse javaBeans components, forward the user to another page, or generate HTML for the java plugin 

    <jsp:action_name  attribute="value>
    
+ jsp:include 
+ jsp:useBean 
+ jsp:setProperty
+ jsp:forward 
    + forward the requester to a new page 
+ jsp:plugin
    + generates browser-specific code that makes an OBJECT or EMBED tag for the java plugin
+ jsp:element 
    + defines XML elements dynamically 
+ jsp:attribute 
    + defines dynamically defined XML element's attribute
+ jsp:body
    + Defines dynamically-defined XML element's body.
+ jsp:text 
    + write template text in JSP pages and documents 

## 2.2 JSP Implicit Objects 
JSP supports some automatically defined variables

+ request 
    + HttpServletRequest object 
+ response 
    + HttpServletResponse object 
+ out 
    + send output  to the client 
+ session 
    + HttpSession object associated with the request 
+ application 
    + The servletContext object associated with the application context 
+ config 
    + servletConfig object associated with the page 
+ pageContext
    + This encapsulates use of server-specific features like higher performance JspWriters.
+ page
    + This is simply a synonym for this, and is used to call the methods defined by the translated servlet class.
+ exception 
    + The Exception object allows the exception data to be accessed by designated JSP. 

