---
title: 'Intigriti January XSS Challenge'
description: 'Challenge writeup'
pubDate: 'jan 16 2025'
heroImage: '/gara.jpg'
pinned: false
---

> TITLE : Intigriti's January Challenge by Godson<br>
> URL : https://challenge-0125.intigriti.io/

![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/janxss/title.jpg)

- It's a simple and interesting challenge that shows how important the basics are in every part of security.

```
--------------------------JS Source--------------------------

        function XSS() {
            return decodeURIComponent(window.location.search).includes('<') || decodeURIComponent(window.location.search).includes('>') || decodeURIComponent(window.location.hash).includes('<') || decodeURIComponent(window.location.hash).includes('>')
        }
        function getParameterByName(name) {
            var url = window.location.href;
            name = name.replace(/[\[\]]/g, "\\$&");
            var regex = new RegExp("[?&]" + name + "(=([^&#]*)|&|#|$)");
            results = regex.exec(url);
            if (!results) return null;
            if (!results[2]) return '';
            return decodeURIComponent(results[2].replace(/\+/g, " "));
        }

        // Function to redirect on form submit
        function redirectToText(event) {
            event.preventDefault();
            const inputBox = document.getElementById('inputBox');
            const text = encodeURIComponent(inputBox.value);
            window.location.href = `/challenge?text=${text}`;
        }

        // Function to display modal if 'text' query param exists
        function checkQueryParam() {
            const text = getParameterByName('text');
            if (text && XSS() === false) {
                const modal = document.getElementById('modal');
                const modalText = document.getElementById('modalText');
                modalText.innerHTML = `Welcome, ${text}!`;
                textForm.remove()
                modal.style.display = 'flex';
            }
        }
```
- Lets try to understand each function and what they are doing to solve the challenge, `checkQueryParam()` function is called while loading the page.

- It checks for the query param `text` and if it exists, it passes the value `text` to `getParameterByName` function.
- The `getParameterByName` function further construct a regex `/[?&]text(=([^&#]*)|&|#|$)/` and executes it on the url `regex.exec(url);`
- The regex meant to match either `?text=somevalue` or `&text=somevalue` and the output is decoded and returned to innerHTML sink.
- There is also `XSS()` function which detect characters `< , >` through `window.location.search` and `window.location.hash`.
- If it founds such characters after decoding `(text && XSS() === false)` this condition will fail and results in failure of our payload entering the sink.
- How do we bypass it? `?text=somevalue` is required to enter the function `checkQueryParam()` and there is proper check on both query param and hash fragment.
- lets understand the structure of URL before solving the challenge.
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/janxss/url.jpg)

> window.location.search - returns the query string from the URL.<br>
> window.location.hash - returns the hash fragment from the URL.

- Both query param and hash has an xss check ! other than that what we can control on the URL?

### The Path:
> தை பிறந்தால் வழி பிறக்கும் 😉

- If we can control the path , we can supply our paylod through `/&text=payload/` and execute xss with it.
- Obviously we can control the path but there is one more problem we need to tackle is to use a non existent path ![ our payload would be an non existent path which will result in 404]
- We can use path traversing and path normalizing to overcome the 404 error.
- Combining everything our final payload looks like this `https://challenge-0125.intigriti.io/&text=%3Cimg%20src='1'%20onerror=alert(document.domain)%3E%2f..%2fchallenge?text=123&text=?text`
- Since the regex match `&text=` our xss payload will be passed to the innerHTML sink [ XSS() function will return false as it searches only the query param and hash fragment of the URL]
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/janxss/xss.png)


