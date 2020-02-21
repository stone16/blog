---
title: PowerMock with EasyMock toturial
date: 2020-02-20 21:11:10
categories: BackEnd
tags:
    - Java
    - Unit Test
    - PowerMock
top:
---
# 1. Intro with an example 

We often find we need to do unit tests for final class, static method, which are not supported by Easymock, Mockito currently. Under such situation, we could use Powermock to help us mock the corresponding classes. 

For detail introduction about powermock, refer to [PowerMock Github](https://github.com/powermock/powermock)

Use example as followed to show how to integrate PowerMock with EasyMock: 

    public final class FinalClassExample {
        
        public String static doNothingStatic() {
            return "test";
        }
    }
    
    @PowerMockIgnore("javax.management.*") // only need when see warning related with jmx or mbeans
    @RunWith(PowerMockRunner.class)  // necessary for powermock 
    @PrepareForTest(FinalClassExample.class)  // necessary for powermock 
    public class FinalClassExampleTest {
        
        
        private IMocksControl control;
        
        @Before
        public void init() {
            // do some initialization here 
            control = EasyMock.createControl();
        }
        
        @Test
        public void test_example() {
            
            PowerMock.mockStatic(FinalClassExample.class);
            
            expect(FinalClassExample.doNothingStatic()).andReturn("test");
            
            PowerMock.replay(FinalClassExample.class);
            
            runYourTest();
            
            PowerMock.verify(FinalClassExample.class);
            
            // do some assertions here
        }
    }
    
# 2. Other APIs 

+ mock final classes or methods 
    + `@RunWith(PowerMockRunner.class)`
    + `@PrepareForTest(ClassWithFinal.class)`
    + `PowerMock.createMock(ClassWithFinal.class); `
    + `PowerMock.replay(mockObject)`
    + `PowerMock.verify(mockObject)`
+ mock private methods 
    + `@RunWith(PowerMockRunner.class)`
    + `@PrepareForTest(ClassWithPrivateMethod.class)`
    + `PowerMock.createPartialMock(ClassWithPrivateMethod.class, "nameOfTheMethodToMock")`
    + Use `PowerMock.expectPrivate(mockObject, "nameOfTheMethodToMock", argument1, argument2)` to expect the method call to `nameOfTheMethodToMock` with arguments `argument1` and `argument2`
    + `PowerMock.replay(mockObject)`
    + `PowerMock.verify(mockObject)`
+ mock construction of new objects 
    + `@RunWith(PowerMockRunner.class)`
    + `@PrepareForTest(ClassThatCreatesTheNewInstance.class)` 
    + `PowerMock.createMock(NewInstanceClass.class)`
    + `PowerMock.expectNew(NewInstanceClass.class).andReturn(mockObject)`
    + `PowerMock.replay(mockObject, NewInstanceClass.class)`
    + `PowerMock.verify(mockObject, NewInstanceClass.class)`
+ mock partial 
    + `@RunWith(PowerMockRunner.class)`
    + `@PrepareForTest(ClassToPartiallyMock.class)`
    + `PowerMock.createPartialMock(ClassToPartiallyMock.class, "nameOfTheFirstMethodToMock", "nameOfTheSecondMethodToMock")`
    + `PowerMock.replay(mockObject)`
    + `PowerMock.verify(mockObject)`

# Reference 
1. https://github.com/powermock/powermock