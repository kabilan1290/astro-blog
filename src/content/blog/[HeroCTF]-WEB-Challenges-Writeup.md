---
title: 'HeroCTF WEB Challenges Writeup'
description: 'ctf writeup'
pubDate: 'Oct 28 2024'
heroImage: '/mickey.jpeg'
---

> <h3>Challenge Name </h3>: Jinjatic
> <h3>Description </h3>: A platform that allows users to render welcome email's template for a given customer, sounds great no ?

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

- User input is accepted as email format and we can supply our payload as `{{7*7}}@gmail.com`