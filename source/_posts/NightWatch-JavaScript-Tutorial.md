---
title: NightWatch JavaScript Tutorial
date: 2020-01-30 22:32:53
categories: FrontEnd
tags:
    - NightWatch
    - FrontEnd
    - JavaScript
    - UI Test
top:
---
# 1. Intro 

+ Complete end-to-end testing solition 
+ written in nodejs and using the WebDriver 
+ WebDriver 
    +  Library for automating web browsers  
    +  A remote control interface that enables intrpspection and controls of user agents. Provides a platform and a restful HTTP api as a way for web browsers to be remotely controlled. 
    
# 2. How to use 

## 2.1 Writing tests in general way

+ use the preferred CSS selector model to locate elements on a page
+ create a separate folder for tests in the project 


E.G we could set couple different steps to interact with the page in multiple ways. 

    module.exports = {
      'step 1' : function (browser) {
        browser
          .url('https://www.google.com')
          .waitForElementVisible('body')
          .setValue('input[type=text]', 'nightwatch')
          .waitForElementVisible('input[name=btnK]')
          .click('input[name=btnK]')
          .pause(1000)
          .assert.containsText('#main', 'Night Watch')
          .end();
      }, 
          'step 2' : function (browser) {
            browser
              .click('input[name=btnK]')
              .pause(1000)
              .assert.containsText('#main', 'Night Watch')
              .end();
          }
    };

## 2.2 Writing tests in ES6 with async/ await

+ improves the readability and ease of writing of tests
+ no longer be available to chain the API commands when using an async function 


    module.exports = {
      'demo test async': async function (browser) {
        // get the available window handles
        const result = await browser.windowHandles();
        console.log('result', result);
    
        // switch to the second window
        // await is not necessary here since we're not interested in the result
        browser.switchWindow(result.value[1]);
      }
    };

## 2.3 Expect Assertions 

    module.exports = {
      'Demo test Google' : function (browser) {
        browser
          .url('https://google.no')
          .pause(1000);
    
        // expect element <body> to be present in 1000ms
        browser.expect.element('body').to.be.present.before(1000);
    
        // expect element <#lst-ib> to have css property 'display'
        browser.expect.element('#lst-ib').to.have.css('display');
    
        // expect element <body> to have attribute 'class' which contains text 'vasq'
        browser.expect.element('body').to.have.attribute('class').which.contains('vasq');
    
        // expect element <#lst-ib> to be an input tag
        browser.expect.element('#lst-ib').to.be.an('input');
    
        // expect element <#lst-ib> to be visible
        browser.expect.element('#lst-ib').to.be.visible;
    
        browser.end();
      }
    };
    
## 2.4 before, after, beforeEach, afterEach 

+ standard hook 
+ before and after will run before and after the test suite
+ beforeEach and afterEach will run before and after each test case 


    module.exports = {
      before : function(browser) {
        console.log('Setting up...');
      },
    
      after : function(browser) {
        console.log('Closing down...');
      },
    
      beforeEach : function(browser) {
    
      },
    
      afterEach : function(browser) {
    
      },
    
      'step one' : function (browser) {
        browser
         // ...
      },
    
      'step two' : function (browser) {
        browser
        // ...
          .end();
      }
    };
    
## 2.5 Controlling the done invocation timeout 

Increase the timeout by defining an `ayncHootTimeout` property in the external globals file. We should set a global module somewhere for all to use 

[globalModule](https://github.com/nightwatchjs/nightwatch/blob/master/examples/globalsModule.js#L20)

## 2.6 Test Env

+ we could define multiple sections of test settings so you could overwrite specific values per env. 


    {
      ...
      "test_settings" : {
        "default" : {
          "launch_url" : "http://localhost",
          "globals" : {
            "myGlobalVar" : "some value",
            "otherGlobal" : "some other value"
          }
        },
    
        "integration" : {
          "launch_url" : "http://staging.host",
          "globals" : {
            "myGlobalVar" : "other value"
          }
        }
      }
    }

## 2.7 Test Groups 

+ we could organize test into groups and run tem as needed.
+ just place them in sub-folders if you want to run them together.

## 2.8 Using Page Objects 

+ Wrap the pages or page fragments of a web app into objects. 
+ Allow a software client to do anything and see anything that a human can be abstracting away the underlying html actions needed to access and manipulate the page 
+ page objects are read from the folder specified in the `page_objects_path` configuration property

## 2.9 Define Elements 

    module.exports = {
      elements: {
        searchBar: {
          selector: 'input[type=text]'
        },
        submit: {
          selector: '//[@name="q"]',
          locateStrategy: 'xpath'
        }
      }
    };
    
+ allow you to refer to the element by its name with an "@" prefix, rather than selector

### 2.9.1 Element Properties 

An element can be specified as an object with `selector` property

+ selector  
+ localeStrategy  'css'
+ index 
    + used to target a specific element in a query that results in multiple elements returned
+ abortOnFailure
+ timeout 
+ retryInterval 
+ suppressNotFoundErrors 

# Reference

1. https://nightwatchjs.org/api/#expect-api
2. https://nightwatchjs.org/guide
