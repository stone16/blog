---
title: CSS pseudo class selectors
date: 2020-01-30 20:54:22
categories: FrontEnd
tags:
    - FrontEnd
    - CSS
top:
---
# 1. What is pseudo selectors? 

> Used to style specified parts of an element: 
> 1. Style the first letter, or line of an element 
> 2. Insert content before, or after the content of an element 

# 2. Why you wanna use it? 

+ Realize some fancy styling, like you want to highlight the first line or first character in an element 
+ You want to insert some common element before/after each specific element


# 3. How to use it? 


    p::first-line {
      color: #ff0000;
      font-variant: small-caps;
    }
    
    p::first-letter {
      color: #ff0000;
      font-size: xx-large;
    }
    
    // you insert gif before every h1
    h1::before {
      content: url(smiley.gif);
    }
    
    h1::after {
      content: url(smiley.gif);
    }
    
    ::selection {
      color: red;
      background: yellow;
    }

# 4. Tips 

## 4.1 Double colon versus single 

The double colon replaced the single-colon notation for pseudo-elements in CSS3. This was an attempt from W3C to distinguish between pseudo-classes and pseudo-elements.

The single-colon syntax was used for both pseudo-classes and pseudo-elements in CSS2 and CSS1.

For backward compatibility, the single-colon syntax is acceptable for CSS2 and CSS1 pseudo-elements.

**Please try to use single instead of double thus it could support almost all browsers **

# Reference 
1. https://www.w3schools.com/css/css_pseudo_elements.asp
2. https://css-tricks.com/almanac/selectors/a/after-and-before/