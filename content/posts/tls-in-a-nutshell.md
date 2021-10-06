---
title: "TLS in a Nutshell"
date: 2021-10-06T02:37:19+01:00
draft: false
cover: https://media.gettyimages.com/photos/close-up-of-lock-and-metal-chain-picture-id84304606?s=612x612
tags: [http,web,internet]
---


# TLS in a nutshell

So it's 1 am once again and I suddenly find myself questioning how the web works from the ground up(not even kidding this time), after roughly 30mins of re validating my knowledge, i have arrived at TLS whilst watching a nice talk by Daniel Stenberg, the author of curl,talk is over [here](https://www.youtube.com/watch?v=pUxyukqoXR4) btw
. In this talk Daniel goes over how the web has evolved from HTTP/1.1 down to HTTP2
and now HTTP 3 which is apparently going to save us all(no offense,i think [QUIC](https://en.wikipedia.org/wiki/QUIC)) is an amazing protocol, but that's besides the point here. 

## So what is my point here?
well every so often I ask myself if I where to explain <insert thing here> to someone would i really be able to do it? if the answer is no , then it's safe to say I don't know enough about said topic, which means I need to do more research or find a way to put words together so the other person can understand. And this was the case with TLS, 
whilst watching Daniel's presentation I honestly never thought about explaining TLS, I mean i've tried once, but i'm quite sure the other person understood only because he was a technical person and not because i did a good job. So now i attempt to explain TLS in as few words as possible. 
	
![quic](https://i.gifer.com/PlA0.gif)
lol get it? quic. Okay enough with the bad jokes. 
	
So you know that little lock icon 🔒  you see in your browser when you head to your favorite website? TLS has something to do with that. Essentially TLS ensures that whoever you want to talk to on the inter webs is who they say they are, This means if i'm going to hipstergram.com, TLS ensures that i am indeed browsing hipstergram.com and what ever I am talking to hipstergram about stays between us, this could be something like login details , password details which would traditionally be visible to someone who is on the same wifi network as me, this is why you get that warning from your browser when you are browsing websites using HTTP, AKA websites without a lock icon. And now you maybe wondering... How?
![how](https://media3.giphy.com/media/l4FGnHKwXZpdYu208/giphy.gif?cid=3640f6095bcf96f74f3847334945c030)
	
To do this TLS uses a combination of symmetric and asymmetric encryption to secure the communication between you and hipstergram, in the case of symmetric encryption me and hipstergram both agree to encrypt our messages using a shared key that has been predetermined ahead of time and this is how we both communicate, but this has one obvious downside if a bad guy somehow finds this key, he can now see all the pictures of my cat i'm uploading and possibly my login information if i'm signing in at that time.  So to solve this problem the first time you visit a webpage your browser verify the webpage using something called the SSL or TLS handshake(you might hear both being used, the main point being TLS is the newer version of SSL) , which is basically a short conversation between your browser and the website to be sure he is who he says he is, it goes a little something like this : 
	
Browser: hey, we need to talk but i'll only speak to you over a secure channel 
	
hipstergram.com: sure, i'll send you my TLS certificate along with my public key 
	
Browser: sure,let me verify that by asking the person who gave you this(certificate authority).
	
Browser: Okay looks good, here's a symmetric encryption key(also called a session key) which you can use to decipher all the messages i send to you. 
	
hipstergram.com: sure!
	
And now you can safely login and like picture of your favorite cat, without prying eyes.
This took me way longer than it should have, because apparently explaining TLS is hard, there's usually too much detail that it becomes overwhelming on or too little that it's unsatisfactory,  so i hope i did a decent job of striking a balance and hopefully you now know a lot more about transport layer security than when you came.

### A bit more technical but great reads 
	
🤩 The original TLS specification: https://datatracker.ietf.org/doc/html/rfc2246#ref-XDR
Obligatory Wikipedia reference: https://en.wikipedia.org/wiki/Transport_Layer_Security
Little more on TLS handshakes: https://www.digicert.com/how-tls-ssl-certificates-work
	



