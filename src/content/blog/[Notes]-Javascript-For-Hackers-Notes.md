---
title: 'Javascript for Hackers - Notes '
description: 'Book notes'
pubDate: 'nov 11 2024'
heroImage: '/hinata.jpeg'
pinned: false
---


# Javascript For hackers notes:

### Hexadecimal:
- Hexadecimal only works on strings and we can't use them as identifiers
` "\x61" - a`
` '\x61' - a`
`function a(){}`
`\x61()` - FAILS

### Unicode:
- Unicode escape works in strings and can be used as identifiers.
`'\u0061'` - a
`"\u0061"` - a
```
>>function a(){
    return 1+2}
>>\u0061()
>>3
```
- We can try create some payloads with the use of unicode
- `\u0061lert()` - alert()
- `\u0061lert(\u0064ocument.domain)` - alert(document.domain)

```
function toUnicode(char) {
    const codePoint = char.charCodeAt(0);
    return '\\u' + codePoint.toString(16).padStart(4, '0');
}
a = "alert"
payload =""
for(let char of a){
    payload += toUnicode(char)
    
}
// '\\u0061\\u006c\\u0065\\u0072\\u0074'

// \u0061\u006c\u0065\u0072\u0074() - `alert()`
```
 - a function to return string to unicode character and we can abuse it to use anywhere as identifier to functioncall.!
 - Unicode can also used inside curly braces!
 `'\u{61}'`
`\u{61}\u{6c}\u{65}\u{72}\u{74}()` - alert()
`"\u{00000000000000000000061}"` - a
- unlimited amount of zero padding and exclusion of zero are allowed.

### Octal:
- Octal escape can be only used inside strings.
`\141` - a
`142` - b 

### Eval and Escapes:
- Eval will decode the strings passed to it and then the engine will execute the decoded string.
- This allow us to bypass the restriction of octal,hex can't be used as identifier.
```
>>eval('\x61=123')
>>a
123
>>eval('\141=1223')
>>a
1223
```
`eval('\\u{61}=100000')` - we might need to double escape the unicode.
