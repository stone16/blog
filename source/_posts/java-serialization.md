---
title: java - serialization
date: 2020-02-05 19:01:26
categories: BackEnd
tags:
    - Java
top:
---
# 1. Overview

An object is eligible for serialization if and only if its class implements the **java.io.Serializable** interface. Serializable is a marker interface (contains no methods) that tell the Java Virtual Machine (JVM) that the objects of this class is ready for being written to and read from a persistent storage or over the network.

# 2. Why we need serialization? 

It's used when the need arises to send data/ object over network or stored in files.

The thing is network and hard disk are hardware component that understand bits and bytes but not Java Objects. 

Serialization is the translation of your Java object's values/states to bytes to send it over network or save it.

Also used to store into database. 

# 3. Implementation 

Ways for serialize/ deserialize  -- xml JSON 

Serialization process is instance independent. 

+ ObjectInputStream 
    + extends java.io.InputStream 
    

    public final Object readObject() throws IOException, ClassNotFoundException;


+ ObjectOutputStream 
    + extends java.io.OutputStream


    public final void writeObject(Object o) throws IOException;


## 3.1 Example 

    public class Person implements Serializable {
        private static final long serialVersionUID = 1L;
        static String country = "ITALY";
        private int age;
        private String name;
        transient int height;
     
        // getters and setters
    }


    @Test
    public void whenSerializingAndDeserializing_ThenObjectIsTheSame() () 
      throws IOException, ClassNotFoundException { 
        Person person = new Person();
        person.setAge(20);
        person.setName("Joe");
         
        FileOutputStream fileOutputStream
          = new FileOutputStream("yourfile.txt");
        ObjectOutputStream objectOutputStream 
          = new ObjectOutputStream(fileOutputStream);
        objectOutputStream.writeObject(person);
        objectOutputStream.flush();
        objectOutputStream.close();
         
        FileInputStream fileInputStream
          = new FileInputStream("yourfile.txt");
        ObjectInputStream objectInputStream
          = new ObjectInputStream(fileInputStream);
        Person p2 = (Person) objectInputStream.readObject();
        objectInputStream.close(); 
      
        assertTrue(p2.getAge() == p.getAge());
        assertTrue(p2.getName().equals(p.getName()));
    }
    
# 4. Caveats 

1. When a class implements the java.io.Serializable interface, all **its sub-classes are serializable as well**.
2. when an object has a reference to another object, these objects must implement the Serializable interface separately, or else a NotSerializableException will be thrown
3. JVM associates a version number with each serializable class. 


Reference

1. https://www.codejava.net/java-se/file-io/why-do-we-need-serialization-in-java
2. https://www.baeldung.com/java-serialization
3. https://www.geeksforgeeks.org/serialization-in-java/