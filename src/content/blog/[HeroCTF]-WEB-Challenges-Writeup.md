---
title: 'HeroCTF WEB Challenges Writeup'
description: 'CTF writeup'
pubDate: 'Oct 28 2024'
heroImage: '/mickey.jpeg'
---

> <h3>Challenge Name :</h3> Jinjatic
> <h3>Description :</h3> A platform that allows users to render welcome email's template for a given customer, sounds great no ?

### Initial analysis:
- From the challenge name and description, it’s clear where this is heading — SSTI!

- The challenge provides an input box for the user’s payload, which then gets rendered. But there’s a twist with how the input field processes data, so let's dive in and analyze the source code.

### src
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
- Using the payload `{{lipsum.__globals__}}@gmail.com`,we were able to read the global objects.

![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/heroctf2024/global.png)

- So possibly we are looking for the end payload `lipsum.__globals__.os.popen('ls').read()` but our payload should need to be in email format which basically rejects `(`,`)`,`[`,`]` making it almost impossible to construct a payload.

### pydantic : email validator

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
