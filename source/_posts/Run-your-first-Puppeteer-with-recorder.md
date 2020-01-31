---
title: Run your first Puppeteer with recorder
date: 2020-01-30 23:16:31
categories: FrontEnd
tags:
    - FrontEnd
    - Puppeteer
    - UI Test
top:
---
This blog wanna give you some introduction on how to write and run your first puppeteer script. 

I love puppeteer over selenium cause it could use js to interact with browser in almost all ways. It's super easy to write some scripts to free you from repetive browser work (click, save, blabla...). I learnt this merely cause I don't want to waste too much of my time on ops work. I believe you guys have the thought with me :) 

# 1. Install puppeteer

In your command line tool, run 

    npm i puppeteer 

It will show you if you have succeeded install puppeteer there. 

# 2. Write some basic script 

Let's write some basic code to interact with one page: 

    > vim firstPuppeteer.js
    
    // in the js file, input 
    
    // Require the node module, our pre installed puppeteer 
    const puppeteer = require('puppeteer');

    (async () => {
    
      // launch a browser 
      const browser = await puppeteer.launch();
      
      // launch a page 
      const page = await browser.newPage();
      
      // Go to some link you wanna go
      await page.goto('https://www.llchen60.com');
    
      // Get the "viewport" of the page, as reported by the page.
      const dimensions = await page.evaluate(() => {
        return {
          width: document.documentElement.clientWidth,
          height: document.documentElement.clientHeight,
          deviceScaleFactor: window.devicePixelRatio
        };
      });
    
      console.log('Dimensions:', dimensions);
    
      await browser.close();
    })();
    
Well, this start script could get the dimension of the page you want. To execute it, run command `node firstPuppeteer.js`

The output would be : `Dimensions: { width: 800, height: 600, deviceScaleFactor: 1 }`

# 3. Write your own interaction with Puppeteer Recorder 

Here, I wanna introduce you a chrome extension named: **Puppeteer Recorder**. Basically, you could start your interaction with browser and this extension can record all your behavior. 

+ It uses ES6 syntax to await for your selected component to be available
+ With recorder, it comes to be super easy to write puppeteer scripts, no more need to inspect all elements in browser and tracing down one by one! 


Still using my blog as an example, I start record when I'm at the home page, and look around, click on the puppeteer turorial link and then try to leave some comments there, the auto generated script are shown below: 

    const puppeteer = require('puppeteer');
    (async () => {
      const browser = await puppeteer.launch()
      const page = await browser.newPage()
      
      let frames = await page.frames()
      const frame_35 = frames.find(f => f.url() === 'https://disqus.com/embed/comments/?base=default&f=leilei-blog&t_u=https%3A%2F%2Fwww.llchen60.com%2FPupperteer-Tutorial%2F&t_d=Pupperteer%20Tutorial%20-%20Leilei%20%7C%20%E7%A3%8A%E7%A3%8A%E7%9A%84%E5%8D%9A%E5%AE%A2&t_t=Pupperteer%20Tutorial%20-%20Leilei%20%7C%20%E7%A3%8A%E7%A3%8A%E7%9A%84%E5%8D%9A%E5%AE%A2&s_o=default#version=50739766d3d12616cb0b6361b7b2fd85')
      const navigationPromise = page.waitForNavigation()
      
      await page.goto('https://llchen60.com/')
      
      await page.setViewport({ width: 1080, height: 1809 })
      
      await page.waitForSelector('.row > .col-lg-8 > .post-preview:nth-child(17) > a > .post-title')
      await page.click('.row > .col-lg-8 > .post-preview:nth-child(17) > a > .post-title')
      
      await navigationPromise
      
      await frame_35.waitForSelector('.postbox > .textarea-outer-wrapper > .textarea-wrapper > div > .placeholder')
      await frame_35.click('.postbox > .textarea-outer-wrapper > .textarea-wrapper > div > .placeholder')
      
      await frame_35.waitForSelector('div #view27_display_name')
      await frame_35.click('div #view27_display_name')
      
      await browser.close()
    })();
    
As you see, it auto generates the code we need, usually we could some modifications and put it on our existing scripts. 

# Reference 
1. https://llchen60.com/Pupperteer-Tutorial/
2. https://chrome.google.com/webstore/search/puppeteer?hl=en-US
3. https://github.com/GoogleChrome/puppeteer
