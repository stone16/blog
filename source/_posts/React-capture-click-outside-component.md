---
title: React - capture click outside component
date: 2020-01-30 23:19:07
categories: FrontEnd
tags:
    - FrontEnd
    - React
top:
---

# 1. Use case 

Suppose you create your own pop up modal, or you invent your own dropdown, you will need to deal/ capture with click outside of the component. This blog will illustrate how to do so. 

# 2. Handy instructions 

## 2.1 Create a ref to div 

    
    render() {
        return (
            <div ref = {node => this.node = node}> </div>
        );
    }
    
## 2.2 Add event listener 

    componentWillMount() {
        document.addEventListener('mousedown', this.handleClick, false);
    }
    
    componentWillUnmount() {
        document.removeEventListener('mousedown', this.handleClick, false);
    }
    
    handleClick = (e) => {
        if (this.node.contains(e.target)) {
            // inside this component, do whatever you want
            return;
        }
        
        // handle outside click
        this.handleOutsideClick();
    }