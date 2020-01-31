---
title: Curl tutorial
date: 2020-01-30 20:41:41
categories: Web
tags:
    - Curl
top:
---
curl = client url, used to make requests to web server in client side. 


#### Send GET request to the url 
    curl https://www.google.com 
    
#### -A Define client's agent header ---- `User-Agent`

    curl -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36' https://google.com

#### -b pass cookie to server 

    curl -b 'foo=bar' https://www.google.com

#### -d send the data body needed when sending POST request 

    curl -d 'login=leilei&pwd=test' -X POST https://www.google.com 
    
    curl -d '@data.txt' -X POST https://www.google.com 
    
#### -e set header's Referer 

    curl -e 'https://google.com?q=example' https://www.example.com

#### --data-urlencode

similar to `-d`, the difference is **it could do URL encode for sent data**

    curl --data-urlencode 'comment=hello world' https://google.com/login

#### -F  send binary file to server 

    curl -F 'file=@photo.png;type=image/png' https://google.com/profile

#### -G create the search string needed

    curl -G -d 'q=kitties' -d 'count=20' https://google.com/search
    
#### -H Add HTTP header

    curl -H 'Accept-Language: en-US' https://google.com

#### --limit-reate 

Limit the speed, simulate env with low speed internet 

    curl --limit-rate 200k https://google.com

#### -u set username and pwd for server authentication 

    curl -u 'bob:12345' https://google.com/login
    
#### -X method for make request 

    curl -X POST https://www.example.com
    
    
# Reference
1. https://www.ruanyifeng.com/blog/2019/09/curl-reference.html
2. https://catonmat.net/cookbooks/curl 
