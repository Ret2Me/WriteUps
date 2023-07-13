+++
title = "SeaEssAreEph"
date = "2023-07-13T00:31:00+02:00"
author = "Filip Poplewski"
authorTwitter = "" #do not include @
cover = "https://mrn00b0t.files.wordpress.com/2021/03/udctf-logo.png"
tags = ["CTF", "Web", "English"]
description = "SeaEssAreEph is a funny WEB challenge where our goal is hack money to buy a flag."
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++


# SeaEssAreEph
SeaEssAreEph is a funny WEB challenge where our goal is hack money to buy a flag.</br></br>Made by Filip "Ret2Me" Poplewski member of WaletSec group
</br>
   

## Beginning
Everything what we have got is website url and port.  
![description](/UDCTF/description.png)
    
   
## Webpage 
Ok so it is bank webpage, lets try find out where  might be a flag.
![page](/UDCTF/page.png)
   
   
Ok i see our goal is hack money to buy a flag for 1337$. 
![buy_flag](/UDCTF/buy_flag.png)
   
   
   
At the website we can find information about CEO, random employee and <b>web admin</b>. In the future it can be used to generate <a href="https://github.com/Mebus/cupp">CUPP</a>.
![workers](/UDCTF/workers.png)
   
   
When we click at Contact button, input named "To" will be filled with Bobby Schnapps username unfortunately when we click Contact to Web Admin input will be empty. 
![contact_us_empty](/UDCTF/contact_us_empty.png)

## Directory scanning
At the beginning I tried scan website in search of hidden administrator panel or other interesting files).
  

![gobuster](/UDCTF/gobuster.png)


As we can see scan doesn't return anything special at all (maybe API can be useful in the future, but it is a directory so right now we can't do with it anything special)
Let's look at what we have got at robots.txt (in this file are stored directories and pages which admin want to hide against the bots)

![robots](/UDCTF/robots.png)
Unfortunately nothing interesting here :(
   
  
   
## Login panel
If user exist and password is incorrect we get error "Bad password".</br>We know that's CEO login was name_surname let's check if web admin login was created in the same way.  
![login](/UDCTF/login.png)
   
   
Yes, we are lucky now we know what is the administrator username, nextly i tried some SQLinjection, brute force, CUPP on CEO and web administrator accounts but nothing works for me.   
   
   
## Source code
Ok so let's create account (it is needed to transfer money what can be very useful) and look at the page source code.

![source](/UDCTF/source.jpg)
   

As we can see webpage doesn't use any csrf-token what is critical vulnerability! We can use it to generate money transaction link which uses GET to send the param. Now we need to send malicious link to web administrator via contact form. 
  
  
## Attack
In the first field we need to fill our username because when website administrator click at the link we want send money to our selfs.</br>At the second field amount of money that will be send to us.
![money_transfer](/UDCTF/money_transfer.png)


"admin" is account created by me. 

![link](/UDCTF/link.png "link")

   
now paste link to message field in "Contact us" form and click send. 
![contact_us](/UDCTF/contact_us.png "contact_us")
    
   
The payload has been shipped lets check account balance.
![sent](/UDCTF/sent.png "sent")
  
  
We are rich, now we can buy a flag :)

![rich](/UDCTF/rich.png "rich")
  
  
  
![flag](/UDCTF/flag.png "flag")
   
   
## End
That was really fun and easy challange were we can learned in practice about how XSRF attack works and why it is so important to use XSRF tokens.
