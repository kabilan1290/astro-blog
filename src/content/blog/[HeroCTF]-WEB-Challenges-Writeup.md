---
title: 'HeroCTF WEB Challenges Writeup'
description: 'CTF writeup'
pubDate: 'Oct 28 2024'
heroImage: '/zoro.jpeg'
---

> <h3>Challenge Name :</h3> Jinjatic
> <h3>Description :</h3> A platform that allows users to render welcome email's template for a given customer, sounds great no ?

### Initial analysis:
- From the challenge name and description, it’s clear where this is heading — SSTI!

- The challenge provides an input box for the user’s payload, which then gets rendered. But there’s a twist with how the input field processes data, so let's dive in and analyze the source code.

### src:
```
#app.py
from flask import Flask, render_template, request
from pydantic import BaseModel, EmailStr, ValidationError
from jinja2 import Template

app = Flask(__name__)

email_template = '''
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Email Result</title>
    <link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-5">
        <div class="alert alert-success text-center">
            <h1>Welcome on the platform !</h1>
            <p>Your email to connect is: <strong>%s</strong></p>
        </div>
        <a href="/mail" class="btn btn-primary">Generate another welcome email</a>
    </div>

    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
'''

class EmailModel(BaseModel):
    email: EmailStr

@app.route('/')
def home():
    return render_template('home.html')

@app.route('/mail')
def mail():
    return render_template('mail.html')

@app.route('/render', methods=['POST'])
def render_email():
    email = request.form.get('email')

    try:
        email_obj = EmailModel(email=email)
        return Template(email_template%(email)).render()
    except ValidationError as e:
        return render_template('mail.html', error="Invalid email format.")

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=80)
```

- User input is accepted as email format and we can supply our payload as `{{7*7}}@gmail.com` ,It will get rendered and return as `49@gmail.com`
- While we try to leverage the template injection to print flask config data through `{{config}}@gmail.com` , it get's failed and we got a empty response `@gmail.com`.
- I tried invoking an error to understand what happened by supplying `{{config|int}}@gmail.com` in my local setup and below is the result.

![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/heroctf2024/undefined.png)

- `config` and other `objects` were not callable in the current context :0
- But confimed builtin filters were working !
- Using the payload `{{lipsum.__globals__}}@gmail.com`,we were able to read the global objects.

![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/heroctf2024/global.png)

- So possibly we are looking for the end payload `lipsum.__globals__.os.popen('ls').read()` but our payload should need to be in email format which basically rejects `(`,`)`,`[`,`]` making it almost impossible to construct a payload.

### pydantic : [email validator]

- The application uses `from pydantic import BaseModel, EmailStr, ValidationError`:pydantic for email validation.
```
class EmailModel(BaseModel):
    email: EmailStr
```
- It uses the above class for creating email model by user supplied string.

`https://github.com/pydantic/pydantic/discussions/2550`

>By default pydantic tries to coerce the input and will hence accept 'Some text `<a@b.com>'` for EmailStr to keep only the email part. What matters is what you want in your inner model !

- Looking at the pydantic documentation `https://docs.pydantic.dev/latest/api/networks/#pydantic.networks.EmailStr`

![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/heroctf2024/confusion.png)

- The discussion gave a limelight that `email: EmailStr` will still accept name string eventhough the document says it had another property at place `NameEmail`.


- Also looking at the documentation `Validate a name and email address combination, as specified by RFC 5322.`

- And as per the RFC , we can use special character `(`,`)` inside the name element of the email using double quotes.

- Hence we can construct our payload like ` "some test payload()" <ab@gmail.com>`.

>`"Some text {{lipsum.__globals__.os.popen('cd ..&&./getflag').read()}}" <ab@gmail.com>`

- The above payload is used to retrieve the flag.

![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/heroctf2024/flag.png)

- That's a Interesting SSTI challenge that requires delving into documentation and understanding the email RFC!

<br>
<hr>

><h3>Challenge Name :</h3> SampleHub
><h3>Description :</h3> Discover SampleHub, the ultimate online repository for accessing a vast collection of sample files. With user-friendly navigation, and robust search features, SampleHub makes it easy to browse, access, and download files efficiently.

### Challenge flow:
- The challenge provides with a search bar where we can search sample files and download it.

```
//Sample files
[
    { "filename": "sample.png", "extension": ".png", "description": "A Portable Network Graphics file that supports lossless data compression. Commonly used for web images and graphics." },
    { "filename": "sample.jpg", "extension": ".jpg", "description": "A JPEG image file, widely used for digital photos and web graphics. It employs lossy compression to reduce file size." },
    { "filename": "sample.pdf", "extension": ".pdf", "description": "A Portable Document Format file used for sharing documents while preserving formatting. It is viewable on any platform using a PDF reader." },
    { "filename": "sample.doc", "extension": ".doc", "description": "A Microsoft Word document file format used for text documents. It supports rich formatting and can include images, tables, and other elements." },
    { "filename": "sample.xls", "extension": ".xls", "description": "A Microsoft Excel spreadsheet file format that supports data analysis and calculation. It allows for complex formulas and data visualization." },
    { "filename": "sample.mp3", "extension": ".mp3", "description": "An audio file format that uses lossy compression to reduce file size. It's commonly used for music and audio streaming." },
    { "filename": "sample.mp4", "extension": ".mp4", "description": "A digital multimedia format commonly used for video files. It supports a wide range of codecs and is ideal for streaming." }
]
```
- We can download the sample file through `/download/:file` ,lets read the source file to know more.
- We only interested in the `app.js` !

```
const express = require("express");
const path    = require("path");

const app  = express();
const PORT = 3000;

app.use(express.static(path.join(__dirname, "public")));
app.set("view engine", "ejs");
app.set("views", path.join(__dirname, "views"));

app.get("/", (req, res) => {
    res.render("index");
});

process.chdir(path.join(__dirname, "samples"));
app.get("/download/:file", (req, res) => {
    const file = path.basename(req.params.file);
    res.download(file, req.query.filename || "sample.png", (err) => {
        if (err) {
            res.status(404).send(`File "${file}" not found`);
        }
    });
});


app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});
```

### Dissecting app.js:
- `process.chdir(path.join(__dirname, "samples"));` changes the current directory to `samples`
- `/download/:file` where user supplied files [`req.params.file`] are passed to `path.basename()`
- We can't do directory traversal due to the nature of `path.basename` working.
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/heroctf2024/path.png)
- If we give payload `/download/../../flag.txt`
```
~/samplehub                          
❯ node
Welcome to Node.js v22.9.0.
Type ".help" for more information.
> path.basename("../../flag.txt")
'flag.txt'
```
- It returns the last portion of a path and trailing directory separators are ignored.
- Bypassing this seems not applicable as there is a url decoder before passing to path.basename function [ which eliminates double encoding and unicode encoding bypasses].

### Hidden in plainsight:
- There is also one more user input is passing to a function but is overlooked.
- `res.download(file, req.query.filename)` , the final file is passed down to res.download and we can control the second parameter? YES !
- Reading the documentation of the `res.download` function.
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/heroctf2024/download.png)
- We can pass down the optional options argument through `/download/something?filename['options']=something` , but there is no defnition as such? how does that work.
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/heroctf2024/option.png)
- If we pass the second argument as `object` it will be interpreted as  options parameter.
- We can now change the `cwd` with the options `root`.
- Our flag is in the format of `.flag.txt` , so we might need the options `dotfiles` and the value as `allow`
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/heroctf2024/format.png)
- The final payload to solve the challenge `/download/.flag.txt?filename[root]=/&filename[dotfiles]=allow`
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/heroctf2024/f.png)

- Thanks for following the writeup until the end! 


