---
title: i18next tutorial
date: 2020-01-30 20:56:25
categories: FrontEnd
tags:
    - FrontEnd
    - i18next
    - JavaScript
top:
---
# 0. Lessons - SkipInterpolation option in i18next-icu 

Overall it works fine for me, one issue I want to highlight is: currently the skipInterpolication option in i18next-icu is not working, insteand, you could leverage on getResource [API](https://www.i18next.com/overview/api#getresource) to get raw data, and define your own substitution logic with regex. [Related github issue](https://github.com/i18next/i18next-icu/issues/11)

# 1. Intro

+ highlights
    + both for front end and back end
        + can be used in nodejs, PHP, IOS, Android, etc. 
    + thorough solution 
        + detect the user language 
        + load the translation 
        + optionally cache the translation 
        + extension support 
            + [personalization](https://www.i18next.com/overview/configuration-options)
    + scalability 
        + we could separate translations into multiple files and load them on demand  

# 2. Starting from Scratch
## 2.1 Basic Example

    // with callback function
    import i18next from 'i18next';
    
    i18next.init({
      lng: 'en',
      debug: true,
      resources: {
        en: {
          translation: {
            "key": "hello world"
          }
        }
      }
    }, function(err, t) {
      // initialized and ready to go!
      document.getElementById('output').innerHTML = i18next.t('key');
    });

    // with promise
    i18next.init({
      lng: 'en',
      debug: true,
      resources: {
        en: {
          translation: {
            "key": "hello world"
          }
        }
      }
    }).then(function(t) {
      // initialized and ready to go!
      document.getElementById('output').innerHTML = i18next.t('key');
    });
    
## 2.2 Configuration

Record options when calling `i18next.init(options, callback)`

+ resources 
+ lng 
+ fallbackLng
    + language to use if translation in user language are not available 
+ whitelist
    + array of allowed languages 
+ ns 
    + string or array of namespaces to load  
+ detection 
    + options for language detection 
+ backend 
    + option for backend 
+ cache 
    + option for cache layer  

# 3. API

## 3.1 General

+ init 
    + `i18next.init(options, callback)`
    + Initialize an i18next instance 
    + callback will be called after all translations were loaded 
+ use 
    + `i18next.use(module)`
    + load additional plugins to i18next 
+ t 
    + `i18next.t(keys, options)`
    +  do translation, we could either use one key as a string or multiple keys as an Array of String. 


    i18next.t('my.key'); // -> will return value in set language
    
    i18next.t(['unknown.key', 'my.key']); // -> will return value for 'my.key' in set language
    
+ exists
    + `i18next.exists(key, options)`
    +  same resolve function as t function, return true if a key exists 
+ getFixedT 
    + `i18next.getFixedT(lng, ns)`
    + return a t function that defaults to given language or namespace 


    // fix language to german
    const de = i18next.getFixedT('de');
    de('myKey');
    
    // or fix the namespace to anotherNamespace
    const anotherNamespace = i18next.getFixedT(null, 'anotherNamespace');
    anotherNamespace('anotherNamespaceKey'); // no need to prefix ns i18n.t('anotherNamespace:anotherNamespaceKey');

+ changeLanguage 
    + `i18next.changeLanguage(lng, callback)`
    + change the language, the callback will be called as soon translation were loaded or an error occurs while loading 
+ languages 
    + `i18next.languages`
    + a set, that will be used to look up the translation value 
+ loadNamespaces
    + `i18next.loadNamespaces(ns, callback)`
    + load additional namespaces not defined in init options 



    i18next.loadNamespaces('myNamespace', (err) => { /* resources have been loaded */ });
    i18next.loadNamespaces(['myNamespace1', 'myNamespace2'], (err) => { /* resources have been loaded */ });
    
    // using Promises
    i18next
      .loadNamespaces(['myNamespace1', 'myNamespace2'])
      .then(() => {});
+ loadLanguages
    + `i18next.loadLanguages(lngs, callbak)`
    + load additional languages not defined in init options 
+ reloadResources 
+ setDefaultNamespace 
    + change the default namespace 

+ dir 
    + `i18next.dir(lng)`
    + return rtl or ltr depending on languages read direction 
+ format 
    + `i18next.format(data, format, lng)`
    + interpolation.format function exposure on init 

## 3.2 Instance Creation 

+ createInstance 
    + `i18next.createInstance(options, callback)`
    + return a new i18next instance 


    const newInstance = i18next.createInstance({
      fallbackLng: 'en',
      ns: ['file1', 'file2'],
      defaultNS: 'file1',
      debug: true
    }, (err, t) => {
      if (err) return console.log('something went wrong loading', err);
      t('key'); // -> same as i18next.t
    }));
    
    // is the same as
    const newInstance = i18next.createInstance();
    newInstance.init({
      fallbackLng: 'en',
      ns: ['file1', 'file2'],
      defaultNS: 'file1',
      debug: true
    }, (err, t) => {
      if (err) return console.log('something went wrong loading', err);
      t('key'); // -> same as i18next.t
    }));

+ cloneInstance 
    + `i18next,cloneInstance(options)`
    + create a clone of the current instance
    + share store, plugins, and initial configuration 
    + can be used to create an instance sharing storage but being independent on set language or default namespaces 


## 3.3 Events 

+ onInitialized 
    + `i18next.on('initialized', function(options){})`
    + get fired after initialization 
+ onLanguageChanged
    + `i18next.on('languageChanged', function(lng){}`
    + get fired when changeLanguage got called 
+ onLoaded 
    + `i18next.on('loaded', function(loaded) {})`
    + get fired when loading resources
+ onFailedLoading
    + `i18next.on('failedLoading', function(lng, ns, msg) {})` 
    + get fired if loading resources failed 
+ onMissingKey
    + `i18next.on('missingKey', function(lngs, namespace, key, res) {})`


## 3.4 Store Events 

+ onAdded
    + `i18next.store.on('added', function(lng, ns) {})` 
+ onRemoved
    + `i18next.store.on('removed', function(lng, ns) {})`


## 3.5 Resource Handling

can be accessed on i18next or i18next.services.resourceStore 

+ getResource 
    + `i18next.getResource(lng, ns, key, options)` 
+ addResource 
    + `i18next.addResource(lng, ns, key, value, options)`
+ addResourceBundle
    + `i18next.addResourceBundle(lng, ns, resources, deep, overwrite)`
    + add a complete bundle 
+ hasResourceBundle 
+ getDataByLanguage
+ getResourceBundle
+ removeResourceBUndle 


# 4. Translation Function 

## 4.1 Essentials 

### 4.1.1 Accessing Keys 

    {
        "key": "value of key",
        "look": {
            "deep": "value of look deep"
        }
    }
    
    // use key for translation, then return its value
    i18next.t('key');
    // -> "value of key"
    
    // key could have its own structure 
    i18next.t('look.deep');
    // -> "value of look deep"
    
### 4.1.2 Passing default value 

    i18next.t('key', 'default value to show')

### 4.1.3 Accessing keys in different namespace 

As mentioned in 5.1, namespace allow you to separate translations into multiple files 


+ init 


    i18next.init({
      ns: ['common', 'moduleA'],
      defaultNS: 'moduleA'
    });
    
    {
        "name": "Module A"
    }

    {
        "button": {
            "save": "save"
        }
    }
    
    i18next.t('name');
    // -> "Module A"
    
    // as shown below, for not default ns, you have to specify the ns name 
    i18next.t('common:button.save');
    // -> "save"


### 4.1.4 Multiple Fallback Keys 

we could call t with an array of key, enable translation with dynamic keys, for a non specific fallback value 

    {
      "error": {
        "unspecific": "Something went wrong.",
        "404": "The page was not found."
      }
    }
    
    // const error = '404';
    i18next.t([`error.${error}`, 'error.unspecific']); // -> "The page was not found"
    
    // const error = '502';
    i18next.t([`error.${error}`, 'error.unspecific']); // -> "Something went wrong"

### 4.1.5 Overview Options 

## 4.2 Interpolation 

It enables you to integrate dynamic values into translation. 

Interpolation values get escaped to save you from possible xss attacks

### 4.2.1 Basic 

Key by default are strings surrounded by curly brackets

    {
        "key": "{{what}} is {{how}}"
    }

    i18next.t('key', { what: 'i18next', how: 'great' });
    // -> "i18next is great"


    // Working with data models
    {
        "key": "I am {{author.name}}"
    }
    
    const author = { 
        name: 'Jan',
        github: 'jamuhl'
    };
    i18next.t('key', { author });
    // -> "I am Jan"

### 4.2.2 Unescape

Per default the values get escaped to save you from possible xss attacks, you could toggle excaping off, by either putting `-` beofre the key, or set the `escapeValue` option to `false` when requesting a translation. 

    {
        "keyEscaped": "no danger {{myVar}}",
        "keyUnescaped": "dangerous {{- myVar}}"
    }
    
    i18next.t('keyEscaped', { myVar: '<img />' });
    // -> "no danger &lt;img &#x2F;&gt;"
    
    i18next.t('keyUnescaped', { myVar: '<img />' });
    // -> "dangerous <img />"
    
    i18next.t('keyEscaped', { myVar: '<img />', interpolation: { escapeValue: false } });
    // -> "no danger <img />" (obviously could be dangerous)

### 4.2.3 Additional Options 

Prefix/ suffix for interpolation and other options can be overridden in the init options or by passing additional options to the `t` function. 

    i18next.init({
        interpolation: { ... }
    });
    
    i18next.t('key', {
        interpolation: { ... }
    });

+ format 
+ formatSeparator 
    + used to separate format from interpolation value 
+ excape 
    + `function excape(str) {return str;} `
+ escapeValue
    + escape passed in values to avoid xss injection 
+ useRawValueToEscape 
    + if true, then value passed into escape function is not casted to string, use with custom escape function that does its own type check 
+ prefix 
+ suffix 
+ prefixEscaped  escaped prefix for interpolation (regexSafe)
+ suffixEscaped  escaped suffix for interpolation (regexSafe)
+ unexcapeSuffix
+ unescapePrefix
+ nestingPrefix
+ nestingSuffix
+ nestingPrefixEscaped 
+ nestingSuffixEscaped
+ defaultVariables
    + global variables to use in interpolation replacements 
+ maxReplaces
    + after how many interpolation runs to break out before throwing a stack overflow  


## 4.3 Formatting 

Format numbers/ dates and you can also use this function for custom formattings. 

### 4.3.1 Basic

    {
        "key": "The current date is {{date, MM/DD/YYYY}}",
        "key2": "{{text, uppercase}} just uppercased"
    }
    
    i18next.init({
        interpolation: {
            format: function(value, format, lng) {
                if (format === 'uppercase') return value.toUpperCase();
                if(value instanceof Date) return moment(value).format(format);
                return value;
            }
        }
    });
    
    i18next.t('key', { date: new Date() });
    // -> "The current date is 07/13/2016"
    
    i18next.t('key2', { text: 'can you hear me' });
    // => "CAN YOU HEAR ME just uppercased"
    
    i18next.on('languageChanged', function(lng) {
      moment.locale(lng);
    });

### 4.3.2 Additional Options 
+ format function 
    + `function format(value, format, lng){}` 
+ formatSeparator
    + used to separate format from interpolation value 

## 4.4 Plurals
i18next support plurals by default. 

    {
      "key": "item",
      "key_plural": "items",
      "keyWithCount": "{{count}} item",
      "keyWithCount_plural": "{{count}} items"
    }
    
    i18next.t('key', {count: 0}); // -> "items"
    i18next.t('key', {count: 1}); // -> "item"
    i18next.t('key', {count: 5}); // -> "items"
    i18next.t('key', {count: 100}); // -> "items"
    i18next.t('keyWithCount', {count: 0}); // -> "0 items"
    i18next.t('keyWithCount', {count: 1}); // -> "1 item"
    i18next.t('keyWithCount', {count: 5}); // -> "5 items"
    i18next.t('keyWithCount', {count: 100}); // -> "100 items"

Usually the plural suffix could be key_plural directly, but you could find more accurate answer [here](https://jsfiddle.net/sm9wgLze)

### 4.4.1 Interval Plurals 

We could define phrases expressing the number of items lies in a range in following ways 

    // add a post processor 
    import i18next from 'i18next';
    import intervalPlural from 'i18next-intervalplural-postprocessor';
    
    i18next
      .use(intervalPlural)
      .init(i18nextOptions);
      
     // define all the keys needed 
    {
      "key1": "{{count}} item",
      "key1_plural": "{{count}} items",
      "key1_interval": "(1){one item};(2-7){a few items};(7-inf){a lot of items};",
      "key2": "{{count}} item",
      "key2_plural": "{{count}} items",
      "key2_interval": "(1){one item};(2-7){a few items};"
    }
    
    // sample - running code 
    i18next.t('key1_interval', {postProcess: 'interval', count: 1}); // -> "one item"
    i18next.t('key1_interval', {postProcess: 'interval', count: 4}); // -> "a few items"
    i18next.t('key1_interval', {postProcess: 'interval', count: 100}); // -> "a lot of items"
    
    // not matching into a range it will fallback to
    // the regular plural form
    i18next.t('key2_interval', {postProcess: 'interval', count: 1}); // -> "one item"
    i18next.t('key2_interval', {postProcess: 'interval', count: 4}); // -> "a few items"
    i18next.t('key2_interval', {postProcess: 'interval', count: 100}); // -> "100 items"

## 4.5 Nesting 

### 4.5.1 Basic 

Allow you to reference other keys in a translation, could be useful to build glossary terms. 

    {
        "nesting1": "1 $t(nesting2)",
        "nesting2": "2 $t(nesting3)",
        "nesting3": "3",
    }
    
    i18next.t('nesting1'); // -> "1 2 3"


we could also reference keys from other namespaces by prepending the namespace

    "nesting1": "1 $t(common:nesting2)"
    
### 4.5.2 Passing Options to nestings 

    {
          "girlsAndBoys": "$t(girls, {'count': {{girls}} }) and {{count}} boy",
          "girlsAndBoys_plural": "$t(girls, {'count': {{girls}} }) and {{count}} boys",
          "girls": "{{count}} girl",
          "girls_plural": "{{count}} girls"
    }
    
    i18next.t('girlsAndBoys', {count: 2, girls: 3});
    // -> "3 girls and 2 boys"


### 4.5.3 Passing nesting to interpolated 

    {
          "key1": "hello world",
          "key2": "say: {{val}}"
    }
    
    i18next.t('key2', {val: '$t(key1)'});
    // -> "say: hello world"

## 4.6 Context 

Differ translations by providing a context  -- useful to provide gender specific translations 
    
    {
          "friend": "A friend",
          "friend_male": "A boyfriend",
          "friend_female": "A girlfriend"
    }
    
    i18next.t('friend'); // -> "A friend"
    i18next.t('friend', { context: 'male' }); // -> "A boyfriend"
    i18next.t('friend', { context: 'female' }); // -> "A girlfriend"


    {
          "friend_male": "A boyfriend",
          "friend_female": "A girlfriend",
          "friend_male_plural": "{{count}} boyfriends",
          "friend_female_plural": "{{count}} girlfriends"
    }
    
    i18next.t('friend', {context: 'male', count: 1}); // -> "A boyfriend"
    i18next.t('friend', {context: 'female', count: 1}); // -> "A girlfriend"
    i18next.t('friend', {context: 'male', count: 100}); // -> "100 boyfriends"
    i18next.t('friend', {context: 'female', count: 100}); // -> "100 girlfriends"
    
## 4.7 Objects and Arrays

You could return objects or arrays to be used by 3rd party modules localization

    {
        "tree": {
            "res": "added {{something}}"
        },
        "array": ['a', 'b', 'c']
    }
    
    i18next.t('tree', { returnObjects: true, something: 'gold' });
    // -> { res: 'added gold' }
    
    i18next.t('array', { returnObjects: true });
    // -> ['a', 'b', 'c']

# 5. Principles 

## 5.1 Namespace 
+ allow you to separate translations thatget loaded into multiple files 
+ separate files could benefits
    + too many segments in a file make you lose the overview
    + not every translation needs to be loaded on the first page, speed up load time 
+ good practice
    + namespace per view/ page
    + namespace per application section 
    + namespace per module which gets lazy loaded 


    i18next.init({
      ns: ['common', 'moduleA', 'moduleB'],
      defaultNS: 'moduleA'
    }, (err, t) => {
      i18next.t('myKey'); // key in moduleA namespace (defined default)
      i18next.t('common:myKey'); // key in common namespace
    });
    
    // load additional namespaces after initialization
    i18next.loadNamespaces('anotherNamespace', (err, t) => { /* ... */ });

## 5.2 Translation Resolution 

A overall process on how i18next attempts to translate your keys into the appropriate content for a given location. 

### 5.2.1 Concepts

+ Keys
    + A key is a specific set to text than provides a corresponding value when look up 
+ Languages 
    + Language to be used for translating a key 
    + If a key is not found, you could gracefully fall back to other languages 

### 5.2.2 Resolution Order 

When translating a key, i18next tries the first combination of **namespace, language, and key**.  If that doesn't work, i18next will try to match with a similar key, looking for a key that best fits the plural form, context and singular form in that order. 

+ similar keys 
    + If the specific key is not found, i18next tries to match the key you are looking for with a similar key, looking for a key that best fits the plural form, context, and singular form in that order.
+ languages
    + i18next will walk through the list of languages, which consists of the current language and the fallback languages.  
+ namespaces
    + walk through current namespaces and the fallback namespaces 
+ fallback keys 
    + if that key is still not found, i18n will walk through the process with the fallbak keys if specified  
+ key not found 
    + will then return the key itself, that being the first key specified if you also specified fallback keys 


## 5.3 Fallback

    {
      "error": {
        "unspecific": "Something went wrong.",
        "404": "The page was not found."
      }
    }
    
    // const error = '404';
    i18next.t([`error.${error}`, 'error.unspecific']) // -> "The page was not found"
    
    // const error = '502';
    i18next.t([`error.${error}`, 'error.unspecific']) // -> "Something went wrong"


# Reference
1. https://www.i18next.com/ 