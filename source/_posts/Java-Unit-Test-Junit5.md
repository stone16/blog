---
title: Java Unit Test - Junit5
date: 2020-02-08 21:12:58
categories: BackEnd
tags:
    - Java
    - Unit Test
top:
---
# 1. Writing Tests

JUnit5 = Junit Platform + Junit Jupiter + Junit Vintage 

First test cases: 

    import static org.junit.jupiter.api.Assertions.assertEquals;
    
    import example.util.Calculator;
    
    import org.junit.jupiter.api.Test;
    
    class MyFirstJUnitJupiterTests {
    
        private final Calculator calculator = new Calculator();
    
        @Test
        void addition() {
            assertEquals(2, calculator.add(1, 1));
        }
    
    }

## 1.1 Annotations 

+ @Test 
    + Denotes that a method is a test method. This annotation does not declare any attributes
+ @ParameterizedTest 
    + Denote a method is a parameterized test 
    + Make it possible to run a test multiple times with different arguments 
    + Must declare at least one source that will provide the arguments for each invocation and then consume the arguments in teh test method 
+ @RepeatedTest 
    + denotes a method is a test template for a repeated test 
    + Provides ability to repeat a test a specific number of times by annotating a method with `@RepeatedTest` and specify the total number of repetitions desired. 
+ @TestFactory 
    + denotes a method is a test factory for dynamic tests
    + dynamic test generated at runtime by a factory method that is annotated with @TestFactory 
    + a factory for test case 
+ @TestTemplate
    + denotes that a method is a template for test cases designed to be invoked multiple times depending on the number of invocation contexts returned by the registered providers. 
+ @TestMethodOrder 
    + Configure the test method execution order 
+ @TestInstance
    + Used to configure the test instance lifecycle for the annotated test class.  
+ @DisplayName
    + Declares a custom display name for the test class or test method.
+ @BeforeEach
    + Denotes that the annotated method should be executed before each @Test, @RepeatedTest, @ParameterizedTest, or @TestFactory method in the current class; analogous to JUnit 4’s @Before. 
+ @AfterEach 
    + Denotes that the annotated method should be executed after each @Test, @RepeatedTest, @ParameterizedTest, or @TestFactory method in the current class; analogous to JUnit 4’s @After
+ @BeforeAll 
    + Denotes that the annotated method should be executed before all @Test, @RepeatedTest, @ParameterizedTest, and @TestFactory methods in the current class; analogous to JUnit 4’s @BeforeClass. Such methods are inherited (unless they are hidden or overridden) and must be static (unless the "per-class" test instance lifecycle is used).
+ @AfterAll 
    + Denotes that the annotated method should be executed after all @Test, @RepeatedTest, @ParameterizedTest, and @TestFactory methods in the current class; analogous to JUnit 4’s @AfterClass. Such methods are inherited (unless they are hidden or overridden) and must be static
+ @Nested 
    + Denotes that the annotated class is a non-static nested test class. @BeforeAll and @AfterAll methods cannot be used directly in a @Nested test class unless the "per-class" test instance lifecycle is used.
+ @Tag
    + Used to declare tags for filtering tests, either at the class or method level; analogous to test groups in TestNG or Categories in JUnit 4. Such annotations are inherited at the class level but not at the method level.
+ @Disabled 
    + Used to disable a test class or test method; analogous to JUnit 4’s @Ignore. Such annotations are not inherited.
+ @ExtendWith
    + Used to register extensions declaratively 
+ @RegisterExtension
    + Used to register extensions programmatically via fields.
+ @TempDir
    + Used to supply a temporary directory via field injection or parameter injection in a lifecycle method or test method 

## 1.2 Test classes and methods 

+ Test classes must not be abstract and mush have a single constructor
+ Test method: any instance method that is directly annotated or meta-annotated with @Test, @RepeatedTest, @ParameterizedTest, @TestFactory, or @TestTemplate
+ Lifecycle Method: any method that is directly annotated or meta-annotated with @BeforeAll, @AfterAll, @BeforeEach, or @AfterEach

A standard unit test class: 

    import static org.junit.jupiter.api.Assertions.fail;
    import static org.junit.jupiter.api.Assumptions.assumeTrue;
    
    import org.junit.jupiter.api.AfterAll;
    import org.junit.jupiter.api.AfterEach;
    import org.junit.jupiter.api.BeforeAll;
    import org.junit.jupiter.api.BeforeEach;
    import org.junit.jupiter.api.Disabled;
    import org.junit.jupiter.api.Test;
    
    class StandardTests {
    
        @BeforeAll
        static void initAll() {
        }
    
        @BeforeEach
        void init() {
        }
    
        @Test
        void succeedingTest() {
        }
    
        @Test
        void failingTest() {
            fail("a failing test");
        }
    
        @Test
        @Disabled("for demonstration purposes")
        void skippedTest() {
            // not executed
        }
    
        @Test
        void abortedTest() {
            assumeTrue("abc".contains("Z"));
            fail("test should have been aborted");
        }
    
        @AfterEach
        void tearDown() {
        }
    
        @AfterAll
        static void tearDownAll() {
        }
    
    }

## 1.3 Assertions 

All assertions are static methods in org.junit.jupiter.api.Assertions class.

See [API doc](https://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assertions.html) for detail: 


+ assertAll()
+ assertArrayEquals()
+ assertArrauEqualsString()
+ assertEquals()
+ assertNotEquals()
+ assertTimeout()
+ assertTimeoutPreemptively()
+ assertTure()
+ fail()


    @Test
        void dependentAssertions() {
            // Within a code block, if an assertion fails the
            // subsequent code in the same block will be skipped.
            assertAll("properties",
                () -> {
                    String firstName = person.getFirstName();
                    assertNotNull(firstName);
    
                    // Executed only if the previous assertion is valid.
                    assertAll("first name",
                        () -> assertTrue(firstName.startsWith("J")),
                        () -> assertTrue(firstName.endsWith("e"))
                    );
                },
                () -> {
                    // Grouped assertion, so processed independently
                    // of results of first name assertions.
                    String lastName = person.getLastName();
                    assertNotNull(lastName);
    
                    // Executed only if the previous assertion is valid.
                    assertAll("last name",
                        () -> assertTrue(lastName.startsWith("D")),
                        () -> assertTrue(lastName.endsWith("e"))
                    );
                }
            );
        }

## 1.4 Assumptions 

Assumptions is a collection of utility methods that support conditional test execution based on assumptions.In direct contrast to failed assertions, failed assumptions do not result in a test failure; rather, a failed assumption results in a **test being aborted**.

+ assumeFalse()
+ assumeTrue()
+ assumingThat(boolean assumption, Executable executable)

     @Test
        void testOnlyOnDeveloperWorkstation() {
            assumeTrue("DEV".equals(System.getenv("ENV")),
                () -> "Aborting test: not on developer workstation");
            // remainder of test
        }
    
        @Test
        void testInAllEnvironments() {
            assumingThat("CI".equals(System.getenv("ENV")),
                () -> {
                    // perform these assertions only on the CI server
                    assertEquals(2, calculator.divide(4, 2));
                });
    
            // perform these assertions in all environments
            assertEquals(42, calculator.multiply(6, 7));
        }

## 1.5 Test Instance Lifecycle 

In order to allow individual test methods to be executed in isolation and to avoid unexpected side effects due to mutable test instance state, **JUnit creates a new instance of each test class before executing each test method**

You can annotate your test class with `@TestInstance(Lifecycle.PER_CLASS)` if you **prefer to execute all test methods on the same test instance**. And if your test mothods rely on state stored in instance variables, may need to reset the state in @BeforeEach or @AfterEach methods. 

The "per-class" mode has some additional benefits over the default "per-method" mode. Specifically, with the "per-class" mode it becomes possible to declare @BeforeAll and @AfterAll on non-static methods as well as on interface default methods. The "per-class" mode therefore also makes it possible to use @BeforeAll and @AfterAll methods in @Nested test classes.

    @TestInstance(Lifecycle.PER_CLASS)
    interface TestLifecycleLogger {
    
        static final Logger logger = Logger.getLogger(TestLifecycleLogger.class.getName());
    
        @BeforeAll
        default void beforeAllTests() {
            logger.info("Before all tests");
        }
    
        @AfterAll
        default void afterAllTests() {
            logger.info("After all tests");
        }
    
        @BeforeEach
        default void beforeEachTest(TestInfo testInfo) {
            logger.info(() -> String.format("About to execute [%s]",
                testInfo.getDisplayName()));
        }
    
        @AfterEach
        default void afterEachTest(TestInfo testInfo) {
            logger.info(() -> String.format("Finished executing [%s]",
                testInfo.getDisplayName()));
        }
    
    }
    
## 1.6 Repeated Test 

    import static org.junit.jupiter.api.Assertions.assertEquals;
    
    import java.util.logging.Logger;
    
    import org.junit.jupiter.api.BeforeEach;
    import org.junit.jupiter.api.DisplayName;
    import org.junit.jupiter.api.RepeatedTest;
    import org.junit.jupiter.api.RepetitionInfo;
    import org.junit.jupiter.api.TestInfo;
    
    class RepeatedTestsDemo {
    
        private Logger logger = // ...
    
        @BeforeEach
        void beforeEach(TestInfo testInfo, RepetitionInfo repetitionInfo) {
            int currentRepetition = repetitionInfo.getCurrentRepetition();
            int totalRepetitions = repetitionInfo.getTotalRepetitions();
            String methodName = testInfo.getTestMethod().get().getName();
            logger.info(String.format("About to execute repetition %d of %d for %s", //
                currentRepetition, totalRepetitions, methodName));
        }
    
        @RepeatedTest(10)
        void repeatedTest() {
            // ...
        }
    
        @RepeatedTest(5)
        void repeatedTestWithRepetitionInfo(RepetitionInfo repetitionInfo) {
            assertEquals(5, repetitionInfo.getTotalRepetitions());
        }
    
        @RepeatedTest(value = 1, name = "{displayName} {currentRepetition}/{totalRepetitions}")
        @DisplayName("Repeat!")
        void customDisplayName(TestInfo testInfo) {
            assertEquals("Repeat! 1/1", testInfo.getDisplayName());
        }
    
        @RepeatedTest(value = 1, name = RepeatedTest.LONG_DISPLAY_NAME)
        @DisplayName("Details...")
        void customDisplayNameWithLongPattern(TestInfo testInfo) {
            assertEquals("Details... :: repetition 1 of 1", testInfo.getDisplayName());
        }
    
        @RepeatedTest(value = 5, name = "Wiederholung {currentRepetition} von {totalRepetitions}")
        void repeatedTestInGerman() {
            // ...
        }
    
    }


## 1.7 Parameterized Tests

With parameterized, we could run a test multiple times with different arguments. And we must declare at least one source that will provide the arguments for each invocation and then consume the arguments in the test method. 

    @ParameterizedTest
    @ValueSource(strings = { "racecar", "radar", "able was I ere I saw elba" })
    void palindromes(String candidate) {
        assertTrue(StringUtils.isPalindrome(candidate));
    }
### 1.7.1 Annotations used in parameterized tests

+ @ValueSource
    + specify a single array of literal values 
+ @NullSource
    + provides a single null argument to the annotated @ParameterizedTest method.
+ @EmptySource
    +  provides a single empty argument to the annotated @ParameterizedTest method for parameters of the following types: java.lang.String, java.util.List, java.util.Set, java.util.Map, primitive arrays (e.g., int[], char[][], etc.), object arrays (e.g.,String[], Integer[][], etc.).
+ @NullAndEmptySource: 
    + a composed annotation that combines the functionality of @NullSource and @EmptySource.
    

    @ParameterizedTest
    @NullAndEmptySource
    @ValueSource(strings = { " ", "   ", "\t", "\n" })
    void nullEmptyAndBlankStrings(String text) {
        assertTrue(text == null || text.trim().isEmpty());
    }
    
+ @EnumSource
    + provides a convenient way to use Enum constants. 
    + also provides an optional names parameter that lets you specify which constants shall be used 


    @ParameterizedTest
    @EnumSource(value = TimeUnit.class, names = { "DAYS", "HOURS" })
    void testWithEnumSourceInclude(TimeUnit timeUnit) {
        assertTrue(EnumSet.of(TimeUnit.DAYS, TimeUnit.HOURS).contains(timeUnit));
    }
    
    @ParameterizedTest
    @EnumSource(value = TimeUnit.class, mode = EXCLUDE, names = { "DAYS", "HOURS" })
    void testWithEnumSourceExclude(TimeUnit timeUnit) {
        assertFalse(EnumSet.of(TimeUnit.DAYS, TimeUnit.HOURS).contains(timeUnit));
        assertTrue(timeUnit.name().length() > 5);
    }


+ @MethodSource
    + Allow you to refer to one or more factory methods of the test class or external classes 
    + Each factory method must generate a stream of arguments


    @ParameterizedTest
    @MethodSource("stringProvider")
    void testWithExplicitLocalMethodSource(String argument) {
        assertNotNull(argument);
    }
    
    static Stream<String> stringProvider() {
        return Stream.of("apple", "banana");
    }

+ @CsvSource 
+ @CsvFileSource
+ @ArgumentsSource 
    + can be used to specify a custom resuable ArgumentsProvider 
    + An implementation of ArgumentsProvider mush be declared as either a top-level class or as a static nested class. 


    @ParameterizedTest
    @ArgumentsSource(MyArgumentsProvider.class)
    void testWithArgumentsSource(String argument) {
        assertNotNull(argument);
    }
    
    public class MyArgumentsProvider implements ArgumentsProvider {
    
        @Override
        public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
            return Stream.of("apple", "banana").map(Arguments::of);
        }
    }

### 1.7.2 Lifecycle and Interoperability 

Each invocation of a parameterized test has the same lifecycle 

## 1.8 Test Templates 

A @TestTemplate method is not a regular test case but rather a template for test cases. It is designed to be invoked multiple times depending on the number of invocation contexts returned by the registerd providers. 

Thus, it must be used in conjunction with a registered TestTemplateInvocationContextProvider extension. Each invocation of a test template method behaves like the execution of a regular @Test method with full support for the same lifecycle callbacks and extensions.

## 1.9 Dynamic Tests 

 @Test describe methods that implement test cases. These test cases are static in the sense that they are fully specified at compile time, and their behavior cannot be changed by anything happening at runtime.
 
 This new kind of test is a dynamic test which is generated at runtime by a factory method that is annotated with @TestFactory
 
 In contrast to @Test methods, a @TestFactory method is not itself a test case but rather a factory for test cases. Thus, a dynamic test is the product of a factory. Technically speaking, a @TestFactory method must return a single DynamicNode or a Stream, Collection, Iterable, Iterator, or array of DynamicNode instances. Instantiable subclasses of DynamicNode are DynamicContainer and 
 
 DynamicContainer instances are composed of **a display name** and **a list of dynamic child nodes**, enabling the creation of arbitrarily nested hierarchies of dynamic nodes. DynamicTest instances will be executed lazily, enabling dynamic and even non-deterministic generation of test cases.


### 1.9.1 Dynamic test lifecycle 

The execution lifecycle of a dynamic test is quite different than it is for a standard @Test case. Specifically, there are **no lifecycle callbacks for individual dynamic tests**. This means that @BeforeEach and @AfterEach methods and their corresponding extension callbacks are** executed for the @TestFactory method** but not for each dynamic test. In other words, if you access fields from the test instance within a lambda expression for a dynamic test, those fields will not be reset by callback methods or extensions between the execution of individual dynamic tests generated by the same @TestFactory method.

## 1.10 Parallel Execution 
### 1.10.1 Mode 
Offers two mode: 

+ SAME_THREAD
    + Force execution in the same thread used by the parent. For example, when used on a test method, the test method will be executed in the same thread as any @BeforeAll or @AfterAll methods of the containing test class. 
+ CONCURRENT
    + Execute concurrently unless a resource lock forces execution in the same thread.

Alternatively, you can use the @Execution annotation to change the execution mode for the annotated element and its subelements (if any) which allows you to activate parallel execution for individual test classes, one by one.

### 1.10.2 Synchronization 

The @ResourceLock annotation allows you to declare that a test class or method uses a specific shared resource that requires synchronized access to ensure reliable test execution. 

    @Execution(CONCURRENT)
    class SharedResourcesDemo {
    
        private Properties backup;
    
        @BeforeEach
        void backup() {
            backup = new Properties();
            backup.putAll(System.getProperties());
        }
    
        @AfterEach
        void restore() {
            System.setProperties(backup);
        }
    
        @Test
        @ResourceLock(value = SYSTEM_PROPERTIES, mode = READ)
        void customPropertyIsNotSetByDefault() {
            assertNull(System.getProperty("my.prop"));
        }
    
        @Test
        @ResourceLock(value = SYSTEM_PROPERTIES, mode = READ_WRITE)
        void canSetCustomPropertyToApple() {
            System.setProperty("my.prop", "apple");
            assertEquals("apple", System.getProperty("my.prop"));
        }
    
        @Test
        @ResourceLock(value = SYSTEM_PROPERTIES, mode = READ_WRITE)
        void canSetCustomPropertyToBanana() {
            System.setProperty("my.prop", "banana");
            assertEquals("banana", System.getProperty("my.prop"));
        }
    
    }

# 2. Extension Model 

In contrast to Runner, TestRule and MethodRule extension points in JUnit4, the JUnit Jupiter extension model consists of a single, coherent concept -- Extension API. 

## 2.1 Registering Extensions 

Extensions can be registered declaraticely via @ExtendWith, programmatically via @RegisterExtension, or automatically via Java's ServiceLoader mechanism. 

### 2.1.1 Declarative Extension Registration 

    @ExtendWith(RandomParametersExtension.class)
    @Test
    void test(@Random int i) {
        // ...
    }

Notice: we could annotate it in class level to register an extension for all tests in this specific class. 

We can also register multiple extensions: 

    @ExtendWith({ DatabaseExtension.class, WebServerExtension.class })
    class MyFirstTests {
        // ...
    }

### 2.1.2 Programmatic Extension Registration 

Register extensions programmatically by annotating fields in test classes with @RegisterExtension 

When an extension is registered declaratively via @ExtendWith, it can typically only be configured via annotations. In contrast, when an extension is registered via @RegisterExtension, it can be **configured programmatically** - for example, in order to pass arguments to the extension’s constructor, a static factory method, or a builder API.

If a @RegisterExtension field is static, the extension will be registered after extensions that are registered at the class level via @ExtendWith. Such static extensions are not limited in which extension APIs they can implement. Extensions registered via static fields may therefore implement class-level and instance-level extension APIs such as BeforeAllCallback, AfterAllCallback, and TestInstancePostProcessor as well as method-level extension APIs such as BeforeEachCallback, etc.

    class WebServerDemo {
    
        @RegisterExtension
        static WebServerExtension server = WebServerExtension.builder()
            .enableSecurity(false)
            .build();
    
        @Test
        void getProductList() {
            WebClient webClient = new WebClient();
            String serverUrl = server.getServerUrl();
            // Use WebClient to connect to web server using serverUrl and verify response
            assertEquals(200, webClient.get(serverUrl + "/products").getResponseStatus());
        }
    
    }


+ For a non-static field with @RegisterExtension, it will be registered after the test class has been instantiated 
+ By default, an instance extension will be registered after extensions that are registered at the method level via @ExtendWith 
+ If the class is configured with @TestInstance(Lifecycle.PER_CLASS) semantics, an instance extension will be registered at the method level via @ExtendWith.

    class DocumentationDemo {
    
        static Path lookUpDocsDir() {
            // return path to docs dir
        }
    
        // The configured DocumentationExtension will be automatically registered as an extension at the method level.
        @RegisterExtension
        DocumentationExtension docs = DocumentationExtension.forPath(lookUpDocsDir());
    
        @Test
        void generateDocumentation() {
            // use this.docs ...
        }
    }

### 2.1.3 Automatic Extension Registration 

JUnit Jupiter also supports global extension registration via Java’s java.util.ServiceLoader mechanism, allowing third-party extensions to be **auto-detected and automatically registered **based on what is available in the classpath.

## 2.2 Conditional Test Execution 

Used to define conditional test execution. 

An ExecutionCondition is evaluated for each container (e.g., a test class) to determine if all the tests it contains should be executed based on the supplied ExtensionContext

## 2.3 Execution Order of User Code and Extensions 

When executing a test class that contains one or more test methods, a number of extension callbacks are called in addition to the user-supplied test and lifecycle methods.

![fig1.png](https://i.loli.net/2020/02/09/9OkZPCtrFL8RfYM.png)

JUnit Jupiter always guarantees wrapping behavior for multiple registered extensions that implement lifecycle callbacks such as BeforeAllCallback, AfterAllCallback, BeforeEachCallback, AfterEachCallback, BeforeTestExecutionCallback, and AfterTestExecutionCallback.

That means that, given two extensions Extension1 and Extension2 with Extension1 registered before Extension2, **any "before" callbacks implemented by Extension1 are guaranteed to execute before any "before" callbacks implemented by Extension2**.


    import static example.callbacks.Logger.afterAllMethod;
    import static example.callbacks.Logger.afterEachMethod;
    import static example.callbacks.Logger.beforeAllMethod;
    import static example.callbacks.Logger.beforeEachMethod;
    
    import org.junit.jupiter.api.AfterAll;
    import org.junit.jupiter.api.AfterEach;
    import org.junit.jupiter.api.BeforeAll;
    import org.junit.jupiter.api.BeforeEach;
    
    /**
     * Abstract base class for tests that use the database.
     */
    abstract class AbstractDatabaseTests {
    
        @BeforeAll
        static void createDatabase() {
            beforeAllMethod(AbstractDatabaseTests.class.getSimpleName() + ".createDatabase()");
        }
    
        @BeforeEach
        void connectToDatabase() {
            beforeEachMethod(AbstractDatabaseTests.class.getSimpleName() + ".connectToDatabase()");
        }
    
        @AfterEach
        void disconnectFromDatabase() {
            afterEachMethod(AbstractDatabaseTests.class.getSimpleName() + ".disconnectFromDatabase()");
        }
    
        @AfterAll
        static void destroyDatabase() {
            afterAllMethod(AbstractDatabaseTests.class.getSimpleName() + ".destroyDatabase()");
        }
    
    }

    
# Reference 
https://junit.org/junit5/docs/current/user-guide/