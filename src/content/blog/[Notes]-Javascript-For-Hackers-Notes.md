---
title: 'Javascript for Hackers - Notes '
description: 'Book notes'
pubDate: 'nov 11 2024'
heroImage: '/hinata.jpeg'
pinned: false
---

### Hexadecimal:
- Hexadecimal only works on strings and we can't use them as identifiers
```
"\x61" - a
'\x61' - a
function a(){}
\x61() - FAILS
```
### Unicode:
- Unicode escape works in strings and can be used as identifiers.
```
'\u0061' - a
"\u0061" - a
```
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
 - `'\u{61}'`
- `\u{61}\u{6c}\u{65}\u{72}\u{74}()` - alert()
- `"\u{00000000000000000000061}"` - a
- unlimited amount of zero padding and exclusion of zero are allowed.

### Octal:
- Octal escape can be only used inside strings.
- `\141` - a
- `\142` - b 

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
- `eval('\\u{61}=100000')` - we might need to double escape the unicode.

### Strings:
- There are three types of strings!
- Single Quoted
- Double Quoted
- Template Strings
```
Code	Result
\b	Backspace
\f	Form Feed
\n	New Line
\r	Carriage Return
\t  Horizontal Tabulator
\v	Vertical Tabulator
\0  Null
```
- We can also escape any character which are not part of escape sequence and it will be treated as actual string
- `"\H\e\l\l\o"` - Hello
- We can use backslash at the end of the line to continue onto the next line.
```
>>>' i continue on to \
    the next line'
result:' i continue on to     the next line'
>>>' i continue on to 
    the next line'
result:Uncaught SyntaxError: Invalid or unexpected token
```
- Templare strings have a feature that allow as to execute arbritrary javascript expressions whithin a placeholder `${}`
```
`${7*7}` - 49
`${`${`${7*7}`}`}` - 49
Template string placeholder itself is supported inside the template string placeholder.
```
- We can simply use template string after a function to call it and it's called tagged template strings.
```
alert`1337` - alert function with 1337 as argument
```
### Call and Apply:
- Call is a property of every function that allows you to call it and change/use the `this.value` of the function in the first argument and any subsequent arguments are passed to that function.

```
>>function x(){
    console.log(this.callmebro)}
>>let hmm = {callmebro:"hmm really?"}
>>x.call(hmm)
hmm really?
----------------------------------------------------
>>function x(){
    console.log(arguments[0])
    console.log(arguments[1])
    console.log(this)
}

>>x.call(null,1,2)

1
2
Window {window: Window, self: Window, document: document, name: '', location: Location, …}
```
- If we do not suply anything to the `this` value in the call function, it will use the window object,the above is the example where we supplied null in the first argument of the call function and it defaults to window object.
- But in strict mode it will be null instead of window object.

```
>>function x(){
    "use strict";
    console.log(this)}
    
>>x.call(null)

null
```
- The apply function is same as call function but we can supply an array of arguments in the second argument.

```
>>function x(){
    console.log(arguments[0])
    console.log(arguments[1])
    console.log(this)
}

>>x.apply(null,[1,2])

1
2
Window {window: Window, self: Window, document: document, name: '', location: Location, …}
```

### Javascript without parentheses:
#### valueOf:
>JavaScript calls the valueOf method to convert an object to a primitive value. You rarely need to invoke the valueOf method yourself; JavaScript automatically invokes it when encountering an object where a primitive value is expected.

```
let obj = {valueOf(){return 1}};
obj+1 //2
```

valueOf is called on almost every object because it needs to extract the primitive value right?
here the above when two value's are getting added and it need to extract the value of `obj` and hence the function will be called and return the value 1 and it will get added to result 2.

what if we abuse the valueOf?
we set the valueOf to alert!

let obj = {valueOf:alert};
obj+1 // Illegal invocationobj+1

It did'nt worked because alert needs `this` object to be window but here it is `obj`

```
valueOf=alert
window+1 // calls alert()
toString=alert
window+'a' // calls alert()
// toString works the same
```

### Calling functions with arguments without parentheses

- The window object has a global handler called onerror `https://developer.mozilla.org/en-US/docs/Web/API/Window/error_event`
```
throw new Error('some new error')
throw 'hey'
```
- The throw statement allow you to specify user defined error object but need not to be an object we can supply as string too.
> below code should be executed on the global context and it will not work on console,we may use this in a online html compiler.
https://stackoverflow.com/questions/26570331/window-onerror-and-javascript-console-errors

```
onerror=alert;throw "hey";
```
- alert box with `Uncaught hey`
```
onerror=eval;
throw"=alert\x28document.domain\x29";
```
- We can use the above payload to execute code and `=` sign is important
- `throw"alert\x28\x29"` will result in `Uncaught alert()` which will not pop up the alert, so we simply assign the uncaught to the alert to make it work.
