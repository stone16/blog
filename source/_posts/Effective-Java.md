---
title: Effective Java
date: 2020-02-04 09:09:44
categories: BackEnd
tags:
    - Java
top:
---

# 1. General Programming 
## 1.1 Scope of Local variables
    for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
        Element e = i.next();
        ... // Do something with e and i
    }
    
    // code with bug
    Iterator<Element> i2 = c2.iterator();
    while (i.hasNext()) {
        Element e2 = i2.next();
        ... // Do something with e2 and i2
    }
    
i - i2. causing a runtime bug. This bug can be caught at compile time if you **use local variable with minimal scope**. 

1. Declare the scope of a local variable where it is first used 
2. initialize every local variable
3. prefer for loops to while loops 
4. keep methods small and focused on a single task 

## 1.2 Use library

Use library will give you some advantages:

+ The knowledge of experts who implemented it and the experience of others who used it before you. 
+ Better performance that is implemented by experts.
+ New features that are added to the libraries in every major release. 

## 1.3 Float, Double and Exact Calculations 

    int getPossibleBoughtItems() {
        double dollarFunds = 1.0;
        int numItems = 0;
        for (double price = 0.10; dollarFunds >= price; price += 0.10) {
            dollarFunds -= price;
            numItems++;
        }
        return numItems;
    }

We should use BigDecimal, int or long for exact calculations. We should avoid using float and double for exact calculations because they are carefully designed for accurate approximations in scientific and engineering calculations. 

Note that the first method above will return 3, which is wrong, with funds left of $0.3999999999999999. The correct return should be 4, with funds left 0 (see the second implementation).

## 1.4 Primitive Types and Boxed Primitives 

When possible, you should use primitives, instead of boxed primitives, because: 

1. Unnecessary use of boxed primitives may result in a hideously slow program because of repeated boxed and unboxed operations. 
2. whenboxes primitives are used, applying `==` operator is almost always wrong and can lead to deadly bugs that are difficult to discover. 


## 1.5 Use of Strings and Other Types 

    // good
    public final class ThreadLocal<T> {
        public ThreadLocal() {};
        public void set (T value) {...};
        public T get() {...};
    }
    // not good 
    public final class ThreadLocal {
        private ThreadLocal() {};
        public static void set (String key, Object value) {...};
        public static Object get(String key) {...};
    }

We should avoid using String, because it is poor substitutes for other value types, or aggregate types, or capacity types. It is cumbersome, slower, error-prone and inflexible than other types. 

## 1.6 String Builder and String Concatenation 

Use String Builder instead of String Concatenation, due to its poor performance. 

    // Good 
    public String firstNamesToString(List<Person> members) {
        StringBuilder sb = new StringBuilder();
        for (Person p : members) {
            sb.append("[");
            sb.append(p.firstName);
            sb.append("]");
        }
        return sb.toString();
    }
    // Bad 
    public String firstNamesToString(List<Person> members) {
        String s = "";
        for (Person p : members) {
            s += "[" + p.firstName + "]";
        }
        return s;
    }
        
## 1.7 Interface and Class reference 

Use of interface reference 

    public List<Person> getPeopleByFirstName(List<Person> members, String firstName) {}
    
It would be desirable, more flexible and more backward-compatible to use interface types to refer to parameters, return values, variables and fields if appropriate interface types exist. If appropriate interface types do not exist, use least specific class types to refer to parameters, return values, variables and fields if appropriate interface types do not exist. 

# 2. Objects 

## 2.1 Static Factory Methods and Constructors 

You are designing a class such as Date and you want to allow a client to obtain an instance of the class, given some input such as instant.

We should use static factory methods because Factory method has following advantages: 

1. have names 
2. not required to create a new object each time invoked 
3. can return an object of any subtype of their return type 
4. decouple service provider frameworks 


## 2.2 Builders and Constructors 

Use builders since it makes code easier for reading

## 2.3 Singleton 

### 2.3.1 Use of enum 

    public enum MySingleton {
        INSTANCE;
    
        public void getDataByMarketplaceId(MarketplaceId id) { ... }
    }

### 2.3.2 Use of static factory 

    public class MySingleton {
        private static final MySingleton INSTANCE = new MySingleton();
        private MySingleton() { ... }
        public static MySingleton getInstance() { return INSTANCE; }
    
        public void getDataByMarketplaceId(MarketplaceId id) { ... }
    }
    

## 2.4 Reusable Objects 

    public class RomanNumber {
        private static Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
        static boolean isRomanNumber(String s) {
            return ROMAN.matcher(s).matches();
        }
    }

Reuse the expensive objects(here is Pattern), in performace critical situations. 

# 3. Classes 

## 3.1 Accessibility of Classes and Members 

Avoid using a public array because a nonzero length array is always mutable, and thus clients will be able to modify the elements of the array. 

## 3.2 Mutability 

In general, classes should be immutable unless there's a very good reason to make them mutable; and if so, you should minimize mutability when designing and implementing classes. Rules to make a class immutable: 

1. Don't provide methods that modify the state of objects that you want to be immutable. 
2. Ensure that the class cann't be extended.
3. Make all fields final to express your intent clearly
4. Make all fields private to prevent clients from obtaining access to mutable objects 
5. Ensure exclusive access to any mutable components


Try to use more immutability, since it offers benefits byu nature: 

1. Immutable objects are simple: providing failure atomicity 
2. Inherently thread-safe, require no synchronization 
3. Can share immutable objects freely
4. Immutable objects make great building blocks for other objects. 


## 3.3 Interfaces and Abstract Classes 

In general, use of interfaces is the best way to define a type that permits multiple implementations because a class can implement multiple interfaces whereas it cannot extend multiple abstract classes. 

## 3.4 Static Member Classes 

    public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
        ...
        static class Node<K,V> implements Map.Entry<K,V> {
            final int hash;
            final K key;
            V value;
            Node<K,V> next;
            ...
            public final K getKey()        { return key; }
            public final V getValue()      { return value; }
            ...
        }
    }
    
A nested class is a class defined within another class, the former should exist only to serve the latter. If you declare a member class that doesn't require access to its enclosing instance then always put static modifier in its declaration. 

If you omit this modifier then each instance will have a hidden extraneous reference to its enclosing instance, and storing this reference takes time and space. More seriously, it can result in the enclosing instance being retained when it would be otherwise be eligible for garbage collection, causing catastrophic memory leak.

# 4. Methods and Generics 

## 4.1 Empty and Null returns 

Never return null in place of an empty array or collection because it will require clients to check null return for all method calls, ugly!

## 4.2 Optional Returns 

    public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
        if (c.isEmpty()) {
            return Optional.empty();
        }
        E result = null;
        for (E e : c) {
            if (result == null || e.compareTo(result) > 0) {
                result = Objects.requireNonNull(e);
            }
        }
        return Optional.of(result);
    }
    
Since Java 8, an Optional-returning method is possible, more flexible, and easier to use than one that throws an exception; it is also less error-prone than one that returns  null . Here are some best practices when using   Optional :

+ Never return a null value from an optinal-returning method because doing so defeats the entire purpose of the facility.
+ Use helpers provided by the facility
    

     String lastWordInLexicon = max(words).orElse("No words..."); ,   
    Toy myToy = max(toys).orElseThrow(ToyException::new); 
+ Container types, including collections, maps, streams, arrays, and optionals, should not be wrapped in optionals, because they have already provided facility to handle empty values.
+ Never return an optional of a boxed primitive type, with possible exception of  Boolean ,  Byte ,   Character ,  Short ,  Float . For other boxed primitive types, use  OptionalInt ,   OptionalLong ,  OptionalDouble  instead.

## 4.3 Generics and Unchecked Warnings 

    public class ArrayList<E> extends AbstractList<E>
            implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
        ...
        public <T> T[] toArray(T[] a) {
            if (a.length < size) {
                @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elementData, size, a.getClass());
                return result;
            }
            System.arraycopy(elementData, 0, a, 0, size);
            if (a.length > size) {
                a[size] = null;
            }
            return a;
        }
        ...
    } 
    
Unchecked warnings are important; don't ignore them because every unchecked warning has the potential to throw a   ClassCastException  at runtime. You should do your best to eliminate as many of them as possible. If you can't, however, get rid of an unchecked warning, but you can prove that the code that provoke it is typesafe then suppress it with the corresponding annotation in the **narrowest possible scope**.

@SupressWarning: An anotation to surpress compile warnings about unchecked generic operations. 

## 4.4 Generics and Wildcards 

    public class Stack<E> {
        ...
        public Stack() { ... }
        public void push(E e) { ... }
        public E pop() { ... }
        public boolean isEmpty { ... }
    
        public void pushAll(Iterable<? extends E> src) {
            for (E e : src) {
                push(e);
            }
        }
    
        public void popAll(Collection<? super E> dst) {
            while (!isEmpty()) {
                dst.add(pop());
            }
        }
    }
    
It is clear that for maximum flexibility you should use wildcard types on input parameters that represent producers or consumers.

+ Remember PECS, which stands for **Producer-Extends and Consumer-Super**. Note that  Comparable  and   Comparator  are always consumers.

# 5. Exceptions 
## 5.1 Checked Exceptions and Unchecked Exceptions

    /**
     * Returns MarketplaceInfo of a given marketplace.
     * @throws NotFoundException if marketplaceId is not found; do not retry.
     * @throws ServiceUnavailableException if MarketplaceService does not respond after 3 retries.
     */
    public MarketplaceInfo getMarketplaceInfoById(MarketplaceId marketplaceId) {
        try {
            return getMarketplaceInfoByIdFromLocalCache(marketplaceId);
        } catch (IOException e) {
            try {
                MarketplaceInfo info = getMarketplaceInfoByIdFromRemoteCache(marketplaceId);
                putMarketplaceInfoToLocalCache(marketplaceId, info);
                return info;
            } catch (IOException e) {
                for (int numRetries = 0; numRetries < 3; numRetries++) {
                    try {
                        // Call dependent service to get marketplace info
                        MarketplaceInfo info = marketplaceService.getMarketplaceInfoById(marketplaceId);
                        putMarketplaceInfoToLocalCache(marketplaceId, info);
                        putMarketplaceInfoToRemoteCache(marketplaceId, info);                
                        return info;
                    } catch (ServiceUnavailableException e) {
                        sleep(5); // sleep 5 seconds before retry
                    } catch (NotFoundException e) {
                        LOG.error("Unable to get marketplace info because marketplace id {} is not found.", marketplaceId);
                        throw e;
                    }
                }
                throw new ServiceUnavailableException("Unable to get marketplace info after 3 retries.");
            }
        }
    }
    
We should: 

+ throw checked exceptions, a subclass of Exception 
    + for recoverable conditions
+ throw unchecked exceptions, a subclass of runtimeException 
    + for programming errors 


When in doubt, throw unchecked exceptions. When throwing checked exceptions, add methods to aid in recovery for clients. 

You should declare checked exceptions individually and document precisely the conditions under which each exception is thrown, by using Javadoc  @throws  tag. If the same exception is thrown by many methods in a class for the same reason then you can document it in the class's documentation comment. In addition, it is particularly important to document unchecked exceptions of methods in interfaces they may throw

## 5.2 Exception Implementation 

+ Provide detail as much as possible
+ detail msg should contain the values of all parameters and fields that have contributed to the exception 
+ No sensitive information contained 


# 6. Lambdas and Streams 

## 6.1 Method references 

+ `Integer::parseInt` -> a static method reference for `str -> Integer.parseInt(str)`
+ `Instant.now()::isAfter` -> a bound method reference for `Instance i = Instant.now(); t -> i.isAfter(t) `
+ `String::toLowerCase` -> an unbound method reference for `str -> str.toLowerCase()`
+ `TreeMap<K, V>::new` -> A class constructor for `() -> new TreeMap<K, V>`

# 7. Concurrency 

## 7.1 Synchronize Access to Sharable Mutable Data 

When multiple threads share mutable data, each thread that reads or writes the data must **perform synchronization**, otherwise there is no guarantee that one thread's changes of the data will be visible to other threads, and therefore may cause liveness and safety failures. These failures are among the most difficult to debug.
`var ++`
It performs two operations on var : (1) it reads the value, (2) it writes back a new value that is equal to the old value plus one. If a second thread reads the field between the time the first thread reads the old value and writes back the new one, then both threads will see the same value and thus return the same serial number, OUCH; this is a safety failure.

Note that the best way to avoid safety failure is **not to share mutable data**, meaning share only immutable data or don't share at all — confine mutable data to a single thread. If you adopt this policy then you should document it carefully, so that the policy is maintained as your program evolves. It is also crucial to have a deep understanding of the frameworks and libraries you're using because they may introduce threads that you are unaware of.
